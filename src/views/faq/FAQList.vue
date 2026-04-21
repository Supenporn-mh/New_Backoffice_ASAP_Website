<template>
  <div class="faq-list">
    <div class="page-header">
      <h1>จัดการคำถามที่พบบ่อย</h1>
      <el-button type="primary" @click="$router.push('/faqs/create')">
        <el-icon><Plus /></el-icon>
        <span style="margin-left: 6px;">เพิ่มคำถาม</span>
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
          <el-input
            v-model="searchQuery"
            placeholder="ค้นหาคำถาม..."
            clearable
            style="width: 200px;"
            :prefix-icon="Search"
          />
        </div>
        <span class="item-count">ทั้งหมด {{ filteredFAQs.length }} รายการ</span>
      </div>

      <el-table
        ref="tableRef"
        v-loading="loading"
        :data="filteredFAQs"
        style="width: 100%"
        row-key="id"
      >

        <el-table-column width="60" align="center">
          <template #default>
            <el-icon class="drag-handle"><Rank /></el-icon>
          </template>
        </el-table-column>

        <el-table-column label="ลำดับ" width="80" align="center">
          <template #default="{ $index }">
            {{ $index + 1 }}
          </template>
        </el-table-column>

        <el-table-column prop="title" label="คำถาม" min-width="300" />

        <el-table-column prop="category" label="หมวดหมู่" width="150" />

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
                @click="$router.push(`/faqs/${row.id}/edit`)"
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
import { Search, Check, Rank, Edit, Delete, Plus, WarningFilled } from '@element-plus/icons-vue'
import Sortable from 'sortablejs'
import { useApi } from '@/composables/useApi'

const { getFAQs, getFAQCategories, deleteFAQ, reorderFAQs } = useApi()

const tableRef = ref(null)

const loading = ref(false)
const faqs = ref([])
const categories = ref([])
const filterCategory = ref('')
const searchQuery = ref('')

const filteredFAQs = computed(() => {
  return faqs.value.filter(faq => {
    const matchCategory = !filterCategory.value || faq.categoryId === filterCategory.value
    const matchSearch = !searchQuery.value ||
      faq.title.toLowerCase().includes(searchQuery.value.toLowerCase())
    return matchCategory && matchSearch
  })
})

const fetchData = async () => {
  loading.value = true
  try {
    const [faqsData, categoriesData] = await Promise.all([
      getFAQs(),
      getFAQCategories()
    ])
    faqs.value = faqsData
    originalItems.value = faqsData.map(f => ({ ...f }))
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
      `คุณต้องการลบคำถาม "${row.title}" ใช่หรือไม่?`,
      'ยืนยันการลบ',
      {
        confirmButtonText: 'ลบ',
        cancelButtonText: 'ยกเลิก',
        type: 'warning'
      }
    )

    await deleteFAQ(row.id)
    ElMessage.success('ลบคำถามสำเร็จ')
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

const hasUnsavedChanges = ref(false)
const originalItems = ref([])

const cancelChanges = () => {
  faqs.value = originalItems.value.map(f => ({ ...f }))
  faqs.value.forEach((f, i) => { f.order = i + 1 })
  hasUnsavedChanges.value = false
}

const savingOrder = ref(false)

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
    const newOrderIds = faqs.value.map(f => f.id)
    await reorderFAQs(newOrderIds)
    ElMessage.success('บันทึกลำดับเรียบร้อย')
    originalItems.value = faqs.value.map(f => ({ ...f }))
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

const initSortable = () => {
  if (!tableRef.value) return
  
  const el = tableRef.value.$el.querySelector('tbody')
  if (!el) return

  Sortable.create(el, {
    handle: '.drag-handle',
    animation: 150,
    onEnd: ({ oldIndex, newIndex }) => {
      if (oldIndex === newIndex) return
      
      const itemMoved = filteredFAQs.value[oldIndex]
      const targetItem = filteredFAQs.value[newIndex]
      
      // Find real indices in source array
      const realOldIndex = faqs.value.findIndex(f => f.id === itemMoved.id)
      const realNewIndex = faqs.value.findIndex(f => f.id === targetItem.id)
      
      const targetRow = faqs.value.splice(realOldIndex, 1)[0]
      faqs.value.splice(realNewIndex, 0, targetRow)

      hasUnsavedChanges.value = true
    }
  })
}
</script>

<style lang="scss" scoped>
.card-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.filters {
  display: flex;
  gap: 12px;
}

.item-count {
  color: #666;
  font-size: 14px;
}

.drag-handle {
  cursor: move;
  color: #999;
}

.table-actions {
  display: flex;
  justify-content: center;
  gap: 4px;
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
