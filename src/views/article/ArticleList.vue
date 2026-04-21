<template>
  <div class="article-list">
    <div class="page-header">
      <h1>จัดการบทความ</h1>
      <el-button type="primary" @click="openStrapiCreate">
        <el-icon><Plus /></el-icon>
        <span style="margin-left: 6px;">เพิ่มบทความ</span>
      </el-button>
    </div>

    <div class="content-card">
      <div class="card-toolbar">
        <div class="filters">
          <el-select
            v-model="filterCategory"
            placeholder="ทุกหมวดหมู่"
            clearable
            style="width: 180px;"
          >
            <el-option
              v-for="cat in categories"
              :key="cat.id"
              :label="cat.name"
              :value="cat.id"
            />
          </el-select>
          <el-select
            v-model="filterStatus"
            placeholder="ทุกสถานะ"
            clearable
            style="width: 140px;"
          >
            <el-option label="เผยแพร่แล้ว" value="published" />
            <el-option label="ฉบับร่าง" value="draft" />
          </el-select>
          <el-input
            v-model="searchQuery"
            placeholder="ค้นหาบทความ..."
            clearable
            style="width: 200px;"
            :prefix-icon="Search"
          />
        </div>
        <span class="item-count">ทั้งหมด {{ filteredArticles.length }} รายการ</span>
      </div>

      <el-table
        ref="tableRef"
        v-loading="loading"
        :data="filteredArticles"
        style="width: 100%"
        row-key="id"
      >
        <el-table-column label="จัดเรียง" width="70" align="center">
          <template #default>
            <el-icon class="drag-handle" style="font-size: 18px;"><Rank /></el-icon>
          </template>
        </el-table-column>

        <el-table-column label="ลำดับ" width="80" align="center">
          <template #default="{ row }">
            <span class="order-text">{{ row.order || '-' }}</span>
          </template>
        </el-table-column>
        <el-table-column label="รูปปก" width="140" align="center">
          <template #default="{ row }">
            <div class="cover-image-wrapper">
              <el-image
                v-if="row.coverImage"
                :src="row.coverImage"
                fit="cover"
                class="cover-image"
                :preview-src-list="[row.coverImage]"
              />
              <div v-else class="cover-placeholder">
                <el-icon :size="24"><Picture /></el-icon>
              </div>
            </div>
          </template>
        </el-table-column>

        <el-table-column prop="title" label="หัวข้อ" min-width="280">
          <template #default="{ row }">
            <div class="article-title">
              <span class="title-text">{{ row.title }}</span>
            </div>
          </template>
        </el-table-column>

        <el-table-column prop="category" label="หมวดหมู่" width="120" />

        <el-table-column label="แท็ก" min-width="200">
          <template #default="{ row }">
            <div class="tags-cell">
              <el-tag
                v-for="tag in (row.tags || []).slice(0, 3)"
                :key="tag"
                size="small"
                type="info"
                class="tag-item"
              >
                {{ tag }}
              </el-tag>
              <el-tag v-if="(row.tags || []).length > 3" size="small" type="info">
                +{{ row.tags.length - 3 }}
              </el-tag>
            </div>
          </template>
        </el-table-column>

        <el-table-column prop="publishedAt" label="วันที่" width="120">
          <template #default="{ row }">
            {{ formatDate(row.publishedAt) }}
          </template>
        </el-table-column>

        <el-table-column prop="status" label="สถานะ" width="120" align="center">
          <template #default="{ row }">
            <span :class="['status-badge', `status-badge--${row.status}`]">
              {{ row.status === 'published' ? 'เผยแพร่' : 'ฉบับร่าง' }}
            </span>
          </template>
        </el-table-column>

        <el-table-column label="จัดการ" width="150" align="center">
          <template #default="{ row }">
            <div class="table-actions">
              <el-button
                type="primary"
                text
                @click="openStrapiEdit(row)"
              >
                <el-icon><Edit /></el-icon>
              </el-button>
              <el-button
                type="danger"
                text
                @click="handleDelete(row)"
              >
                <el-icon><Delete /></el-icon>
              </el-button>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- Bottom Action Bar -->
    <transition name="bar-slide">
      <div v-if="hasUnsavedChanges" class="bottom-action-bar">
        <div class="bar-info">
          <el-icon class="bar-info__icon"><WarningFilled /></el-icon>
          <span>มีการเปลี่ยนแปลงลำดับที่ยังไม่ได้บันทึก</span>
        </div>
        <div class="bar-actions">
          <button class="btn-cancel" :disabled="savingOrder" @click="cancelChanges">
            ยกเลิก
          </button>
          <button class="btn-save" :disabled="savingOrder" @click="saveOrder">
            <span v-if="savingOrder" class="btn-save__spinner" />
            <el-icon v-else><Check /></el-icon>
            <span>{{ savingOrder ? 'กำลังบันทึก...' : 'บันทึกลำดับ' }}</span>
          </button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { Search, Picture, Rank, Check, Plus, Edit, Delete, WarningFilled } from '@element-plus/icons-vue'
import Sortable from 'sortablejs'
import { useApi } from '@/composables/useApi'

// Strapi Admin URL
const STRAPI_ADMIN_URL = import.meta.env.VITE_STRAPI_API_URL || 'https://my-strapi-api-0fz5.onrender.com'

const { getArticles, getArticleCategories, deleteArticle, reorderArticles } = useApi()

const tableRef = ref(null)
const hasUnsavedChanges = ref(false)
const originalItems = ref([])

const cancelChanges = () => {
  articles.value = originalItems.value.map(a => ({ ...a }))
  articles.value.forEach((a, i) => { a.order = i + 1 })
  hasUnsavedChanges.value = false
}
const savingOrder = ref(false)

const loading = ref(false)
const articles = ref([])
const categories = ref([])
const filterCategory = ref('')
const filterStatus = ref('')
const searchQuery = ref('')

// Open Strapi Admin to create new article
const openStrapiCreate = () => {
  window.open(`${STRAPI_ADMIN_URL}/admin/content-manager/collection-types/api::article.article/create`, '_blank')
}

// Open Strapi Admin to edit article
const openStrapiEdit = (row) => {
  // Use documentId for Strapi v5, fallback to id for older versions
  const identifier = row.documentId || row.id
  window.open(`${STRAPI_ADMIN_URL}/admin/content-manager/collection-types/api::article.article/${identifier}`, '_blank')
}

const formatDate = (date) => {
  if (!date) return '-'
  return new Date(date).toLocaleDateString('th-TH', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric'
  })
}

const filteredArticles = computed(() => {
  return articles.value.filter(article => {
    const matchCategory = !filterCategory.value || article.categoryId === filterCategory.value
    const matchStatus = !filterStatus.value || article.status === filterStatus.value
    const matchSearch = !searchQuery.value ||
      article.title.toLowerCase().includes(searchQuery.value.toLowerCase())
    return matchCategory && matchStatus && matchSearch
  })
})

const fetchData = async () => {
  loading.value = true
  try {
    const [articlesData, categoriesData] = await Promise.all([
      getArticles(),
      getArticleCategories()
    ])
    articles.value = articlesData
    originalItems.value = articlesData.map(a => ({ ...a }))
    categories.value = categoriesData
  } catch (error) {
    ElMessage.error('ไม่สามารถโหลดข้อมูลได้')
  } finally {
    loading.value = false
  }
}

const handleDelete = async (row) => {
  try {
    await ElMessageBox.confirm(
      `คุณต้องการลบบทความ "${row.title}" ใช่หรือไม่?`,
      'ยืนยันการลบ',
      {
        confirmButtonText: 'ลบ',
        cancelButtonText: 'ยกเลิก',
        type: 'warning'
      }
    )

    await deleteArticle(row.id)
    ElMessage.success('ลบบทความสำเร็จ')
    fetchData()
  } catch (error) {
    if (error !== 'cancel') {
      ElMessage.error('ไม่สามารถลบได้')
    }
  }
}

onMounted(() => {
  fetchData().then(() => {
    nextTick(() => {
      initSortable()
    })
  })
})

const initSortable = () => {
  if (!tableRef.value) return
  
  const el = tableRef.value.$el.querySelector('tbody')
  if (!el) return

  Sortable.create(el, {
    handle: '.drag-handle',
    animation: 150,
    onEnd: ({ oldIndex, newIndex }) => {
      if (oldIndex === newIndex) return

      const targetRow = articles.value.splice(oldIndex, 1)[0]
      articles.value.splice(newIndex, 0, targetRow)
      
      articles.value.forEach((a, index) => {
        a.order = index + 1
      })

      hasUnsavedChanges.value = true
    }
  })
}

const saveOrder = async () => {
  try {
    await ElMessageBox.confirm(
      'บันทึกลำดับการแสดงผลใหม่?',
      'ยืนยัน',
      {
        confirmButtonText: 'บันทึก',
        cancelButtonText: 'ยกเลิก',
        distinguishCancelAndClose: true
      }
    )
  } catch {
    return
  }
  savingOrder.value = true
  try {
    const newOrderIds = articles.value.map(a => a.id)
    await reorderArticles(newOrderIds)
    ElMessage.success('บันทึกลำดับเรียบร้อย')
    originalItems.value = articles.value.map(a => ({ ...a }))
    hasUnsavedChanges.value = false
  } catch (error) {
    ElMessageBox.alert('ทำรายการไม่สำเร็จ กรุณาลองใหม่อีกครั้ง', 'แจ้งเตือน', {
      type: 'error',
      confirmButtonText: 'ตกลง'
    })
    fetchData()
  } finally {
    savingOrder.value = false
  }
}
</script>

<style lang="scss" scoped>
.card-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  flex-wrap: wrap;
  gap: 12px;
}

.drag-handle {
  cursor: move;
  color: #999;
}

.filters {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.item-count {
  color: #666;
  font-size: 14px;
}

// Cover Image Styles
.cover-image-wrapper {
  width: 80px;
  height: 50px;
  border-radius: 6px;
  overflow: hidden;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.08);
  background: #f5f7fa;
  margin: 0 auto;
}

.cover-image {
  width: 100%;
  height: 100%;
  display: block;
  transition: transform 0.3s ease;

  &:hover {
    transform: scale(1.05);
  }
}

.cover-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #c0c4cc;
  background: linear-gradient(135deg, #f5f7fa 0%, #e4e8eb 100%);
}

// Article Title
.article-title {
  .title-text {
    font-weight: 500;
    color: #1D2433;
    line-height: 1.5;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }
}

// Table Enhancements
:deep(.el-table) {
  --el-table-border-color: #f0f2f5;

  .el-table__row {
    transition: background-color 0.2s ease;

    &:hover > td {
      background-color: #fafbfc !important;
    }
  }

  .el-table__header th {
    background-color: #fafbfc;
    font-weight: 600;
    color: #606266;
    font-size: 13px;
  }

  .el-table__cell {
    padding: 8px 0;
  }
}

.table-actions {
  display: flex;
  justify-content: center;
  gap: 4px;

  .el-button {
    padding: 8px;

    &:hover {
      background-color: rgba(0, 0, 0, 0.04);
      border-radius: 6px;
    }
  }
}

.tags-cell {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;

  .tag-item {
    margin: 0;
    border-radius: 4px;
  }
}

// Status Badge Improvements
.order-text {
  font-weight: 600;
  color: #606266;
}

.status-badge {
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 500;


  &--draft {
    background-color: #fff1f0;
    color: #ff4d4f;
  }
}

.bottom-action-bar {
  position: fixed;
  bottom: 0;
  left: 260px;
  right: 0;
  z-index: 99;
  background: #fff;
  border-top: 1px solid #E5E7EB;
  box-shadow: 0 -2px 12px rgba(0, 0, 0, 0.06);

  .bar-info {
    display: flex;
    align-items: center;
    gap: 6px;
    padding: 7px 32px;
    background: #FFFBEB;
    border-bottom: 1px solid #FDE68A;
    font-size: 13px;
    color: #92400E;

    &__icon {
      font-size: 14px;
      color: #D97706;
      flex-shrink: 0;
    }
  }

  .bar-actions {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 14px 32px;
  }
}

.btn-cancel {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 9px 24px;
  border: 1px solid #E5E7EB;
  border-radius: 8px;
  background: #fff;
  color: #6B7280;
  font-size: 14px;
  font-weight: 500;
  font-family: inherit;
  cursor: pointer;
  transition: all 0.15s;

  &:hover:not(:disabled) {
    border-color: #9CA3AF;
    background: #F9FAFB;
    color: #374151;
  }

  &:disabled {
    opacity: 0.45;
    cursor: not-allowed;
  }
}

.btn-save {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 9px 28px;
  border: none;
  border-radius: 8px;
  background: #FF595A;
  color: #fff;
  font-size: 14px;
  font-weight: 600;
  font-family: inherit;
  cursor: pointer;
  box-shadow: 0 1px 3px rgba(255, 89, 90, 0.3), 0 1px 2px rgba(255, 89, 90, 0.2);
  transition: all 0.15s;

  &:hover:not(:disabled) {
    background: #E54849;
    box-shadow: 0 4px 10px rgba(255, 89, 90, 0.35);
    transform: translateY(-1px);
  }

  &:active:not(:disabled) {
    transform: translateY(0);
    box-shadow: 0 1px 3px rgba(255, 89, 90, 0.25);
  }

  &:disabled {
    opacity: 0.65;
    cursor: not-allowed;
    transform: none;
  }

  &__spinner {
    width: 14px;
    height: 14px;
    border: 2px solid rgba(255, 255, 255, 0.35);
    border-top-color: #fff;
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
    flex-shrink: 0;
  }
}

.bar-slide-enter-active,
.bar-slide-leave-active {
  transition: transform 0.25s cubic-bezier(0.4, 0, 0.2, 1), opacity 0.25s ease;
}

.bar-slide-enter-from,
.bar-slide-leave-to {
  transform: translateY(100%);
  opacity: 0;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>
