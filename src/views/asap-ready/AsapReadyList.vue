<template>
  <div class="asap-ready-list">
    <div class="page-header">
      <h1>ASAP พร้อมให้บริการ</h1>
      <el-button type="primary" @click="$router.push('/asap-ready/create')">
        <el-icon><Plus /></el-icon>
        <span style="margin-left: 6px;">เพิ่มรูปภาพ</span>
      </el-button>
    </div>

    <div class="content-card">
      <div class="card-toolbar">
        <span class="item-count">ทั้งหมด {{ items.length }} รายการ</span>
      </div>

      <el-table
        ref="tableRef"
        v-loading="loading"
        :data="items"
        style="width: 100%"
        row-key="id"
      >
        <el-table-column width="60" align="center">
          <template #default>
            <el-icon class="drag-handle"><Rank /></el-icon>
          </template>
        </el-table-column>

        <el-table-column prop="order" label="ลำดับ" width="80" align="center" />

        <el-table-column label="รูปภาพ (Desktop / TH)" width="200">
          <template #default="{ row }">
            <el-image
              :src="row.imageDesktop?.th || row.imageDesktop"
              fit="cover"
              style="width: 160px; height: 60px; border-radius: 4px;"
            >
              <template #error>
                <div class="image-placeholder">ไม่มีรูป</div>
              </template>
            </el-image>
          </template>
        </el-table-column>

        <el-table-column prop="title" label="ชื่อ (Admin)" min-width="200" />

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

        <el-table-column label="จัดการ" width="130" align="center">
          <template #default="{ row }">
            <div class="table-actions">
              <el-button
                type="primary"
                text
                @click="$router.push(`/asap-ready/${row.id}/edit`)"
              >
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

    <div v-if="hasUnsavedChanges" class="floating-save">
      <el-button type="success" size="large" :loading="savingOrder" @click="saveOrder">
        <el-icon><Check /></el-icon>
        <span style="margin-left: 8px;">บันทึกลำดับ</span>
      </el-button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { Plus, Edit, Delete, Rank, Check } from '@element-plus/icons-vue'
import Sortable from 'sortablejs'
import { useApi } from '@/composables/useApi'

const { getAsapReadyItems, updateAsapReadyItem, deleteAsapReadyItem, reorderAsapReadyItems } = useApi()

const tableRef = ref(null)
const loading = ref(false)
const items = ref([])
const hasUnsavedChanges = ref(false)
const savingOrder = ref(false)

const fetchItems = async () => {
  loading.value = true
  try {
    items.value = await getAsapReadyItems()
  } catch {
    ElMessage.error('ไม่สามารถโหลดข้อมูลได้')
  } finally {
    loading.value = false
  }
}

const handleStatusChange = async (row) => {
  try {
    await updateAsapReadyItem(row.id, { isActive: row.isActive })
    ElMessage.success('อัปเดตสถานะสำเร็จ')
  } catch {
    ElMessage.error('ไม่สามารถอัปเดตสถานะได้')
    row.isActive = !row.isActive
  }
}

const handleDelete = async (row) => {
  try {
    await ElMessageBox.confirm(
      `คุณต้องการลบ "${row.title}" ใช่หรือไม่?`,
      'ยืนยันการลบ',
      { confirmButtonText: 'ลบ', cancelButtonText: 'ยกเลิก', type: 'warning' }
    )
    await deleteAsapReadyItem(row.id)
    ElMessage.success('ลบสำเร็จ')
    fetchItems()
  } catch (error) {
    if (error !== 'cancel') ElMessage.error('ไม่สามารถลบได้')
  }
}

const saveOrder = async () => {
  savingOrder.value = true
  try {
    const newOrderIds = items.value.map(i => i.id)
    await reorderAsapReadyItems(newOrderIds)
    ElMessage.success('บันทึกลำดับเรียบร้อย')
    hasUnsavedChanges.value = false
  } catch {
    ElMessageBox.alert('ทำรายการไม่สำเร็จ กรุณาลองใหม่', 'แจ้งเตือน', {
      type: 'error', confirmButtonText: 'ตกลง'
    })
    fetchItems()
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
      const moved = items.value.splice(oldIndex, 1)[0]
      items.value.splice(newIndex, 0, moved)
      items.value.forEach((item, index) => { item.order = index + 1 })
      hasUnsavedChanges.value = true
    }
  })
}

onMounted(() => {
  fetchItems().then(() => nextTick(() => initSortable()))
})
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

.drag-handle {
  cursor: move;
  color: #999;
  &:hover { color: #666; }
}

.table-actions {
  display: flex;
  justify-content: center;
  gap: 4px;
}

.image-placeholder {
  width: 160px;
  height: 60px;
  background: #f5f5f5;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #ccc;
  font-size: 11px;
}

.floating-save {
  position: fixed;
  bottom: 30px;
  right: 30px;
  z-index: 999;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  border-radius: 4px;
  animation: slideUp 0.3s ease-out;
}

@keyframes slideUp {
  from { transform: translateY(20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}
</style>
