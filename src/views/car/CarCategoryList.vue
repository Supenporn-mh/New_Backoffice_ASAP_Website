<template>
  <div class="car-category-list">
    <!-- Sticky Header -->
    <div class="sticky-header">
      <div class="page-header">
        <h1>จัดการประเภทกลุ่มรถ</h1>
        <el-button type="primary" @click="openDialog()">
          <el-icon><Plus /></el-icon>
          <span style="margin-left: 6px;">เพิ่มประเภทกลุ่มรถ</span>
        </el-button>
      </div>

      <!-- Stats Summary -->
      <div class="stats-row">
        <div class="stat-item">
          <span class="stat-value">{{ categories.length }}</span>
          <span class="stat-label">ประเภททั้งหมด</span>
        </div>
        <div class="stat-item">
          <span class="stat-value">{{ activeCount }}</span>
          <span class="stat-label">เปิดใช้งาน</span>
        </div>
        <div class="stat-item">
          <span class="stat-value">{{ totalCars }}</span>
          <span class="stat-label">รถทั้งหมด</span>
        </div>
      </div>
    </div>

    <!-- Table -->
    <div class="content-card">
      <el-table ref="tableRef" v-loading="loading" :data="categories" style="width: 100%" row-key="id">

        <el-table-column width="50" align="center">
          <template #default>
            <el-icon class="drag-handle"><Rank /></el-icon>
          </template>
        </el-table-column>

        <el-table-column prop="order" label="ลำดับ" width="80" align="center" />

        <el-table-column label="รูปภาพ" width="120" align="center">
          <template #default="{ row }">
            <el-image
              v-if="row.image"
              :src="row.image"
              fit="cover"
              style="width: 72px; height: 52px; border-radius: 6px;"
              :preview-src-list="[row.image]"
            />
            <div v-else class="image-empty">
              <el-icon><Picture /></el-icon>
            </div>
          </template>
        </el-table-column>

        <el-table-column prop="name" label="ชื่อประเภทกลุ่มรถ" min-width="180" />

        <el-table-column label="Sippcode" min-width="200">
          <template #default="{ row }">
            <div class="sippcode-badges">
              <code
                v-for="(code, idx) in (row.sippcodes || [])"
                :key="idx"
                class="sippcode-badge"
              >{{ code }}</code>
              <span v-if="!row.sippcodes?.length" class="text-muted">-</span>
            </div>
          </template>
        </el-table-column>

        <el-table-column label="จำนวนรุ่น" width="110" align="center">
          <template #default="{ row }">
            <span class="car-count">
              <el-icon><Van /></el-icon>
              {{ row.carCount || 0 }} รุ่น
            </span>
          </template>
        </el-table-column>

        <el-table-column label="แสดงผลบนเว็บ" width="130" align="center">
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

        <el-table-column label="จัดการ" width="120" align="center">
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

    <!-- Bottom Action Bar -->
    <transition name="bar-slide">
      <div v-if="hasUnsavedChanges" class="bottom-action-bar">
        <div class="bar-info">
          <el-icon class="bar-info__icon"><WarningFilled /></el-icon>
          <span>มีการเปลี่ยนแปลงลำดับที่ยังไม่ได้บันทึก</span>
        </div>
        <div class="bar-actions">
          <button class="btn-cancel" :disabled="savingOrder" @click="cancelChanges">ยกเลิก</button>
          <button class="btn-save" :disabled="savingOrder" @click="saveOrder">
            <span v-if="savingOrder" class="btn-save__spinner" />
            <el-icon v-else><Check /></el-icon>
            <span>{{ savingOrder ? 'กำลังบันทึก...' : 'บันทึกลำดับ' }}</span>
          </button>
        </div>
      </div>
    </transition>

    <!-- Dialog -->
    <el-dialog
      v-model="dialogVisible"
      :title="editingCategory ? 'แก้ไขประเภทกลุ่มรถ' : 'เพิ่มประเภทกลุ่มรถ'"
      width="600px"
    >
      <el-form
        ref="formRef"
        :model="form"
        :rules="rules"
        label-position="top"
      >
        <el-form-item label="รูปภาพประเภทกลุ่มรถ" prop="image">
          <div class="image-upload-wrapper">
            <el-upload
              class="image-uploader"
              :show-file-list="false"
              :auto-upload="false"
              :on-change="handleImageChange"
              accept="image/*"
            >
              <div v-if="form.image" class="uploaded-image">
                <el-image :src="form.image" fit="contain" />
                <div class="image-actions">
                  <el-button type="danger" size="small" @click.stop="removeImage">
                    <el-icon><Delete /></el-icon>
                    <span>ลบรูป</span>
                  </el-button>
                </div>
              </div>
              <div v-else class="upload-placeholder">
                <el-icon class="upload-icon"><Plus /></el-icon>
                <span>อัปโหลดรูปภาพ</span>
                <p class="upload-hint">แนะนำขนาด 400x300 px</p>
              </div>
            </el-upload>
          </div>
        </el-form-item>

        <el-row :gutter="16">
          <el-col :span="16">
            <el-form-item label="ชื่อประเภทกลุ่มรถ" prop="name">
              <el-input
                v-model="form.name"
                placeholder="เช่น Economy, SUV, EV Car"
              />
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="ลำดับ" prop="order">
              <el-input-number v-model="form.order" :min="1" style="width: 100%;" />
            </el-form-item>
          </el-col>
        </el-row>

        <el-form-item label="Sippcode" required>
          <el-select
            v-model="form.sippcodes"
            multiple
            filterable
            placeholder="เลือก Sippcode"
            style="width: 100%;"
          >
            <el-option
              v-for="item in sippcodeOptions"
              :key="item"
              :label="item"
              :value="item"
            />
          </el-select>
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
        <el-button type="primary" :loading="saving" @click="handleSubmit">
          <el-icon><Check /></el-icon>
          <span>บันทึก</span>
        </el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, nextTick } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { useApi } from '@/composables/useApi'
import { Check, Van, Edit, Delete, Picture, Plus, Rank, WarningFilled } from '@element-plus/icons-vue'
import Sortable from 'sortablejs'

const {
  getCarCategories,
  createCarCategory,
  updateCarCategory,
  deleteCarCategory,
  reorderCarCategories
} = useApi()

const loading = ref(false)
const saving = ref(false)
const dialogVisible = ref(false)
const categories = ref([])
const editingCategory = ref(null)
const formRef = ref()
const tableRef = ref()
const sippcodeOptions = ['ECAR', 'CCAR', 'SCAR', 'IFAR', 'EVAR', 'HCAR', 'HCMR', 'LCAR', 'PCAR', 'FCAR', 'PFAR', 'LFAR', 'SFAR']

const activeCount = computed(() => categories.value.filter(c => c.isActive).length)
const totalCars = computed(() => categories.value.reduce((sum, c) => sum + (c.carCount || 0), 0))

const form = reactive({
  name: '',
  sippcodes: [],
  image: '',
  order: 1,
  isActive: true
})

const rules = {
  name: [{ required: true, message: 'กรุณากรอกชื่อประเภทกลุ่มรถ', trigger: 'blur' }],
  order: [{ required: true, message: 'กรุณากรอกลำดับ', trigger: 'blur' }]
}


const handleImageChange = (file) => {
  const reader = new FileReader()
  reader.onload = (e) => {
    form.image = e.target.result
  }
  reader.readAsDataURL(file.raw)
}

const removeImage = () => {
  form.image = ''
}

const hasUnsavedChanges = ref(false)
const savingOrder = ref(false)
const originalItems = ref([])

const cancelChanges = () => {
  categories.value = originalItems.value.map(c => ({ ...c }))
  categories.value.forEach((c, i) => { c.order = i + 1 })
  hasUnsavedChanges.value = false
}

const saveOrder = async () => {
  savingOrder.value = true
  try {
    const newOrderIds = categories.value.map(c => c.id)
    await reorderCarCategories(newOrderIds)
    ElMessage.success('บันทึกลำดับเรียบร้อย')
    originalItems.value = categories.value.map(c => ({ ...c }))
    hasUnsavedChanges.value = false
  } catch {
    ElMessage.error('ทำรายการไม่สำเร็จ กรุณาลองใหม่')
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
      const moved = categories.value.splice(oldIndex, 1)[0]
      categories.value.splice(newIndex, 0, moved)
      categories.value.forEach((c, i) => { c.order = i + 1 })
      hasUnsavedChanges.value = true
    }
  })
}

const fetchCategories = async () => {
  loading.value = true
  try {
    categories.value = await getCarCategories()
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
    Object.assign(form, {
      name: category.name,
      sippcodes: category.sippcodes?.length ? [...category.sippcodes] : [''],
      image: category.image,
      order: category.order,
      isActive: category.isActive
    })
  } else {
    form.name = ''
    form.sippcodes = []
    form.image = ''
    form.order = categories.value.length + 1
    form.isActive = true
  }
  dialogVisible.value = true
}

const handleSubmit = async () => {
  if (!formRef.value) return

  await formRef.value.validate(async (valid) => {
    if (valid) {
      saving.value = true
      try {
        if (editingCategory.value) {
          await updateCarCategory(editingCategory.value.id, form)
          ElMessage.success('แก้ไขประเภทกลุ่มรถสำเร็จ')
        } else {
          await createCarCategory(form)
          ElMessage.success('สร้างประเภทกลุ่มรถสำเร็จ')
        }
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
    await updateCarCategory(row.id, { isActive: row.isActive })
    ElMessage.success('อัปเดตสถานะสำเร็จ')
  } catch (error) {
    ElMessage.error('ไม่สามารถอัปเดตสถานะได้')
    row.isActive = !row.isActive
  }
}

const handleDelete = async (row) => {
  try {
    await ElMessageBox.confirm(
      `คุณต้องการลบประเภทกลุ่มรถ "${row.name}" ใช่หรือไม่?`,
      'ยืนยันการลบ',
      {
        confirmButtonText: 'ลบ',
        cancelButtonText: 'ยกเลิก',
        type: 'warning'
      }
    )

    await deleteCarCategory(row.id)
    ElMessage.success('ลบประเภทกลุ่มรถสำเร็จ')
    fetchCategories()
  } catch (error) {
    if (error !== 'cancel') {
      ElMessage.error('ไม่สามารถลบได้')
    }
  }
}

onMounted(() => {
  fetchCategories().then(() => nextTick(() => initSortable()))
})
</script>

<style lang="scss" scoped>
.car-category-list {
  .page-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 24px;

    h1 {
      margin: 0;
      font-size: 24px;
      font-weight: 600;
    }
  }
}

// Sticky Header
.sticky-header {
  position: sticky;
  top: 0;
  z-index: 100;
  background: #F9F9F9;
  margin-left: -24px;
  margin-right: -24px;
  padding: 24px 24px 24px 24px;
  margin-bottom: 24px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
}

// Stats Row
.stats-row {
  display: flex;
  gap: 16px;
  margin-bottom: 24px;

  .stat-item {
    background: #fff;
    border-radius: 12px;
    padding: 16px 24px;
    display: flex;
    flex-direction: column;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);

    .stat-value {
      font-size: 28px;
      font-weight: 700;
      color: #FF595A;
    }

    .stat-label {
      font-size: 13px;
      color: #666;
      margin-top: 4px;
    }
  }
}

.image-empty {
  width: 72px;
  height: 52px;
  border-radius: 6px;
  background: #F3F4F6;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #9CA3AF;
  margin: 0 auto;
  font-size: 20px;
}

.sippcode-badges {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.sippcode-badge {
  background: #e8f5e9;
  padding: 3px 8px;
  border-radius: 4px;
  font-size: 11px;
  color: #2e7d32;
  font-weight: 500;
}

.car-count {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: 13px;
  color: #666;

  .el-icon { color: #FF595A; }
}

.text-muted {
  color: #9CA3AF;
  font-size: 13px;
}

.table-actions {
  display: flex;
  justify-content: center;
  gap: 4px;
}

// Dialog styles
.image-upload-wrapper {
  width: 100%;
}

.image-uploader {
  width: 100%;

  :deep(.el-upload) {
    width: 100%;
  }
}

.uploaded-image {
  width: 100%;
  height: 200px;
  position: relative;
  border: 1px solid #dcdfe6;
  border-radius: 12px;
  overflow: hidden;
  background: #fafafa;

  .el-image {
    width: 100%;
    height: 100%;
  }

  .image-actions {
    position: absolute;
    bottom: 12px;
    right: 12px;
  }
}

.upload-placeholder {
  width: 100%;
  height: 200px;
  border: 2px dashed #dcdfe6;
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #999;
  cursor: pointer;
  transition: all 0.3s;

  &:hover {
    border-color: #FF595A;
    color: #FF595A;
  }

  .upload-icon {
    font-size: 40px;
    margin-bottom: 8px;
  }

  .upload-hint {
    font-size: 12px;
    margin-top: 8px;
    color: #bbb;
  }
}


.drag-handle {
  cursor: move;
  color: #9CA3AF;
  &:hover { color: #6B7280; }
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

    &__icon { font-size: 14px; color: #D97706; flex-shrink: 0; }
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

  &:hover:not(:disabled) { border-color: #9CA3AF; background: #F9FAFB; color: #374151; }
  &:disabled { opacity: 0.45; cursor: not-allowed; }
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
  box-shadow: 0 1px 3px rgba(255, 89, 90, 0.3);
  transition: all 0.15s;

  &:hover:not(:disabled) { background: #E54849; transform: translateY(-1px); }
  &:active:not(:disabled) { transform: translateY(0); }
  &:disabled { opacity: 0.65; cursor: not-allowed; transform: none; }

  &__spinner {
    width: 14px; height: 14px;
    border: 2px solid rgba(255,255,255,0.35);
    border-top-color: #fff;
    border-radius: 50%;
    animation: spin 0.7s linear infinite;
    flex-shrink: 0;
  }
}

.bar-slide-enter-active, .bar-slide-leave-active {
  transition: transform 0.25s cubic-bezier(0.4, 0, 0.2, 1), opacity 0.25s ease;
}
.bar-slide-enter-from, .bar-slide-leave-to {
  transform: translateY(100%);
  opacity: 0;
}

@keyframes spin { to { transform: rotate(360deg); } }

// Button icon spacing
:deep(.el-button) {
  display: inline-flex;
  align-items: center;
  gap: 6px;

  .el-icon { margin: 0; }
}
</style>
