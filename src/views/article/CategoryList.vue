<template>
  <div class="category-list">
    <div class="page-header">
      <h1>หมวดหมู่บทความ</h1>
      <el-button type="primary" @click="openDialog()">
        <el-icon><Plus /></el-icon>
        <span style="margin-left: 6px;">เพิ่มหมวดหมู่</span>
      </el-button>
    </div>

    <div class="content-card">
      <el-table
        ref="tableRef"
        v-loading="loading"
        :data="categories"
        style="width: 100%"
        row-key="id"
      >

        <el-table-column width="60" align="center">
          <template #default>
            <el-icon class="drag-handle"><Rank /></el-icon>
          </template>
        </el-table-column>

        <el-table-column prop="order" label="ลำดับ" width="80" align="center" />

        <el-table-column prop="name" label="ชื่อหมวดหมู่" min-width="200" />

        <el-table-column prop="slug" label="Slug" min-width="150">
          <template #default="{ row }">
            <code>{{ row.slug }}</code>
          </template>
        </el-table-column>

        <el-table-column prop="isActive" label="แสดงผลบนเว็บ" width="130" align="center">
          <template #default="{ row }">
            <el-switch
              v-model="row.isActive"
              inline-prompt
              active-text="แสดง"
              inactive-text="ซ่อน"
              @change="handleStatusChange(row)"
            />
          </template>
        </el-table-column>

        <el-table-column label="จัดการ" width="150" align="center">
          <template #default="{ row }">
            <div class="table-actions">
              <el-button type="primary" text @click="openDialog(row)">
                <el-icon><Edit /></el-icon>
              </el-button>
              <el-button type="danger" text @click="handleDelete(row)">
                <el-icon><Delete /></el-icon>
              </el-button>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <!-- Dialog -->
    <el-dialog
      v-model="dialogVisible"
      :title="editingCategory ? 'แก้ไขหมวดหมู่' : 'เพิ่มหมวดหมู่'"
      width="500px"
    >
      <el-form
        ref="formRef"
        :model="form"
        :rules="rules"
        label-position="top"
      >
        <el-form-item label="ชื่อหมวดหมู่" prop="name">
          <el-input
            v-model="form.name"
            placeholder="กรอกชื่อหมวดหมู่"
            @blur="generateSlug"
          />
        </el-form-item>

        <el-form-item label="Slug" prop="slug">
          <el-input v-model="form.slug" placeholder="auto-generate" />
        </el-form-item>

        <el-form-item label="ลำดับ" prop="order">
          <el-input-number v-model="form.order" :min="1" style="width: 100%;" />
        </el-form-item>

        <el-form-item label="สถานะ">
          <el-switch
            v-model="form.isActive"
            inline-prompt
            active-text="แสดง"
            inactive-text="ซ่อน"
          />
        </el-form-item>
      </el-form>

      <template #footer>
        <el-button @click="dialogVisible = false">ยกเลิก</el-button>
        <el-button type="primary" :loading="saving" :disabled="!isDirtyDialog || saving" @click="handleSubmit">
          บันทึก
        </el-button>
      </template>
    </el-dialog>
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
import { ref, reactive, onMounted, nextTick, watch } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { Plus, Edit, Delete, Rank, Check, WarningFilled } from '@element-plus/icons-vue'
import Sortable from 'sortablejs'
import { useApi } from '@/composables/useApi'

const {
  getArticleCategories,
  createArticleCategory,
  updateArticleCategory,
  deleteArticleCategory,
  reorderArticleCategories
} = useApi()

const tableRef = ref(null)

const loading = ref(false)
const saving = ref(false)
const isDirtyDialog = ref(false)
const dialogVisible = ref(false)
const categories = ref([])
const editingCategory = ref(null)
const formRef = ref()

const form = reactive({
  name: '',
  slug: '',
  order: 1,
  isActive: true
})

watch(() => form, () => { isDirtyDialog.value = true }, { deep: true })

const rules = {
  name: [{ required: true, message: 'กรุณากรอกชื่อหมวดหมู่', trigger: 'blur' }],
  slug: [{ required: true, message: 'กรุณากรอก Slug', trigger: 'blur' }],
  order: [{ required: true, message: 'กรุณากรอกลำดับ', trigger: 'blur' }]
}

const generateSlug = () => {
  if (form.name && !form.slug) {
    form.slug = form.name
      .toLowerCase()
      .replace(/[^\w\sก-๙]/g, '')
      .replace(/\s+/g, '-')
  }
}

const fetchCategories = async () => {
  loading.value = true
  try {
    categories.value = await getArticleCategories()
    originalItems.value = categories.value.map(c => ({ ...c }))
  } catch (error) {
    ElMessage.error('ไม่สามารถโหลดข้อมูลได้')
  } finally {
    loading.value = false
  }
}

const openDialog = (category = null) => {
  editingCategory.value = category
  if (category) {
    Object.assign(form, category)
  } else {
    form.name = ''
    form.slug = ''
    form.order = categories.value.length + 1
    form.isActive = true
  }
  nextTick(() => { isDirtyDialog.value = false })
  dialogVisible.value = true
}

const handleSubmit = async () => {
  if (!formRef.value) return

  await formRef.value.validate(async (valid) => {
    if (valid) {
      saving.value = true
      try {
        if (editingCategory.value) {
          await updateArticleCategory(editingCategory.value.id, form)
          ElMessage.success('แก้ไขหมวดหมู่สำเร็จ')
        } else {
          await createArticleCategory(form)
          ElMessage.success('สร้างหมวดหมู่สำเร็จ')
        }
        isDirtyDialog.value = false
        dialogVisible.value = false
        fetchCategories()
      } catch (error) {
        ElMessage.error('เกิดข้อผิดพลาด')
      } finally {
        saving.value = false
      }
    }
  })
}

const handleStatusChange = async (row) => {
  try {
    await updateArticleCategory(row.id, { isActive: row.isActive })
    ElMessage.success('อัปเดตสถานะสำเร็จ')
  } catch (error) {
    ElMessage.error('ไม่สามารถอัปเดตสถานะได้')
    row.isActive = !row.isActive
  }
}

const handleDelete = async (row) => {
  try {
    await ElMessageBox.confirm(
      `คุณต้องการลบหมวดหมู่ "${row.name}" ใช่หรือไม่?`,
      'ยืนยันการลบ',
      {
        confirmButtonText: 'ลบ',
        cancelButtonText: 'ยกเลิก',
        type: 'warning'
      }
    )

    await deleteArticleCategory(row.id)
    ElMessage.success('ลบหมวดหมู่สำเร็จ')
    fetchCategories()
  } catch (error) {
    if (error !== 'cancel') {
      ElMessage.error('ไม่สามารถลบได้')
    }
  }
}

onMounted(() => {
  fetchCategories().then(() => {
    nextTick(() => {
      initSortable()
    })
  })
})

const hasUnsavedChanges = ref(false)
const originalItems = ref([])

const cancelChanges = () => {
  categories.value = originalItems.value.map(c => ({ ...c }))
  categories.value.forEach((c, i) => { c.order = i + 1 })
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
    const newOrderIds = categories.value.map(c => c.id)
    await reorderArticleCategories(newOrderIds)
    ElMessage.success('บันทึกลำดับเรียบร้อย')
    originalItems.value = categories.value.map(c => ({ ...c }))
    hasUnsavedChanges.value = false
  } catch (error) {
    ElMessageBox.alert('ทำรายการไม่สำเร็จ กรุณาลองใหม่อีกครั้ง', 'แจ้งเตือน', {
      type: 'error',
      confirmButtonText: 'ตกลง'
    })
    fetchCategories()
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

      const targetRow = categories.value.splice(oldIndex, 1)[0]
      categories.value.splice(newIndex, 0, targetRow)
      
      categories.value.forEach((c, index) => {
        c.order = index + 1
      })

      hasUnsavedChanges.value = true
    }
  })
}
</script>

<style lang="scss" scoped>
.drag-handle {
  cursor: move;
  color: #999;
}

.table-actions {
  display: flex;
  justify-content: center;
  gap: 4px;
}

code {
  background: #f5f5f5;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 13px;
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
