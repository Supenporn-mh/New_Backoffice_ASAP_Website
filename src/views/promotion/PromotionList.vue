<template>
  <div class="promotion-list">
    <div class="page-header">
      <h1>จัดการโปรโมชัน</h1>
      <el-button
        type="primary"
        :disabled="promotions.length >= 3"
        @click="$router.push('/promotions/create')"
      >
        <el-icon><Plus /></el-icon>
        <span style="margin-left: 6px;">เพิ่มโปรโมชัน</span>
      </el-button>
    </div>

    <div class="content-card">
      <div class="card-toolbar">
        <span class="item-count">ทั้งหมด {{ promotions.length }}/3 รายการ</span>
        <el-alert
          v-if="promotions.length >= 3"
          title="ถึงจำนวนสูงสุดแล้ว (3 โปรโมชัน)"
          type="warning"
          :closable="false"
          show-icon
          style="width: auto;"
        />
      </div>

      <el-table
        ref="tableRef"
        v-loading="loading"
        :data="promotions"
        style="width: 100%"
        row-key="id"
      >
        <el-table-column width="60" align="center">
          <template #default>
            <el-icon class="drag-handle"><Rank /></el-icon>
          </template>
        </el-table-column>

        <el-table-column prop="order" label="ลำดับ" width="80" align="center" />

        <el-table-column label="รูปภาพ" width="180">
          <template #default="{ row }">
            <el-image
              :src="row.image"
              fit="cover"
              style="width: 140px; height: 100px; border-radius: 4px;"
            />
          </template>
        </el-table-column>

        <el-table-column prop="title" label="ชื่อโปรโมชัน" min-width="180" />

        <el-table-column label="ระยะเวลา" width="200">
          <template #default="{ row }">
            <div class="date-range">
              <span>{{ formatDate(row.startDate) }}</span>
              <span class="date-separator">ถึง</span>
              <span>{{ formatDate(row.endDate) }}</span>
            </div>
          </template>
        </el-table-column>

        <el-table-column label="สถานะ" width="120" align="center">
          <template #default="{ row }">
            <span :class="['status-badge', `status-badge--${getStatus(row)}`]">
              {{ getStatusText(row) }}
            </span>
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
              <el-button
                type="primary"
                text
                @click="$router.push(`/promotions/${row.id}/edit`)"
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
import { ref, onMounted, nextTick } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { Plus, Edit, Delete, Rank, Check, WarningFilled } from '@element-plus/icons-vue'
import Sortable from 'sortablejs'
import { useApi } from '@/composables/useApi'

const { getPromotions, updatePromotion, deletePromotion, reorderPromotions } = useApi()

const tableRef = ref(null)

const loading = ref(false)
const promotions = ref([])

const formatDate = (date) => {
  if (!date) return '-'
  return new Date(date).toLocaleDateString('th-TH', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric'
  })
}

const getStatus = (row) => {
  if (!row.isActive) return 'inactive'
  const now = new Date()
  const start = new Date(row.startDate)
  const end = new Date(row.endDate)
  if (now < start) return 'draft'
  if (now > end) return 'expired'
  return 'active'
}

const getStatusText = (row) => {
  const status = getStatus(row)
  const map = {
    active: 'กำลังแสดง',
    draft: 'รอแสดง',
    expired: 'หมดอายุ',
    inactive: 'ปิดใช้งาน'
  }
  return map[status]
}

const fetchPromotions = async () => {
  loading.value = true
  try {
    promotions.value = await getPromotions()
    originalItems.value = promotions.value.map(p => ({ ...p }))
  } catch (error) {
    ElMessage.error('ไม่สามารถโหลดข้อมูลได้')
  } finally {
    loading.value = false
  }
}

const handleStatusChange = async (row) => {
  try {
    await updatePromotion(row.id, { isActive: row.isActive })
    ElMessage.success('อัปเดตสถานะสำเร็จ')
  } catch (error) {
    ElMessage.error('ไม่สามารถอัปเดตสถานะได้')
    row.isActive = !row.isActive
  }
}

const handleDelete = async (row) => {
  try {
    await ElMessageBox.confirm(
      `คุณต้องการลบโปรโมชัน "${row.title}" ใช่หรือไม่?`,
      'ยืนยันการลบ',
      {
        confirmButtonText: 'ลบ',
        cancelButtonText: 'ยกเลิก',
        type: 'warning'
      }
    )

    await deletePromotion(row.id)
    ElMessage.success('ลบโปรโมชันสำเร็จ')
    fetchPromotions()
  } catch (error) {
    if (error !== 'cancel') {
      ElMessage.error('ไม่สามารถลบได้')
    }
  }
}

onMounted(() => {
  fetchPromotions().then(() => {
    nextTick(() => {
      initSortable()
    })
  })
})

const hasUnsavedChanges = ref(false)
const originalItems = ref([])

const cancelChanges = () => {
  promotions.value = originalItems.value.map(p => ({ ...p }))
  promotions.value.forEach((p, i) => { p.order = i + 1 })
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
    const newOrderIds = promotions.value.map(p => p.id)
    await reorderPromotions(newOrderIds)
    ElMessage.success('บันทึกลำดับเรียบร้อย')
    originalItems.value = promotions.value.map(p => ({ ...p }))
    hasUnsavedChanges.value = false
  } catch (error) {
    ElMessageBox.alert('ทำรายการไม่สำเร็จ กรุณาลองใหม่อีกครั้ง', 'แจ้งเตือน', {
      type: 'error',
      confirmButtonText: 'ตกลง'
    })
    fetchPromotions()
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

      const targetRow = promotions.value.splice(oldIndex, 1)[0]
      promotions.value.splice(newIndex, 0, targetRow)
      
      promotions.value.forEach((p, index) => {
        p.order = index + 1
      })

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

.item-count {
  color: #666;
  font-size: 14px;
}

.date-range {
  font-size: 13px;

  .date-separator {
    margin: 0 4px;
    color: #999;
  }
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
