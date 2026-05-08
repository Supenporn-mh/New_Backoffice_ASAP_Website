<template>
  <div class="car-featured-sorting">
    <div class="sticky-header">
      <div class="page-header">
        <h1>การแสดงผลรถยนต์</h1>
      </div>
    </div>

    <div class="content-card">
      <el-tabs v-model="activeTab" class="feature-tabs">

        <!-- ดีลที่ดีที่สุด -->
        <el-tab-pane name="bestDeals">
          <template #label>
            <span class="tab-label">
              <el-icon><Discount /></el-icon>
              ดีลที่ดีที่สุด
            </span>
          </template>

          <div class="tab-description">
            เลือกรุ่นรถหรือ SIPP Code ที่จะแสดงในส่วน <strong>"ดีลที่ดีที่สุด"</strong> บนหน้าเว็บ และลากเรียงลำดับได้ตามต้องการ
          </div>

          <div class="list-toolbar">
            <span class="item-count">{{ bestDeals.length }} รายการ</span>
            <el-button type="primary" @click="openAddDialog('bestDeals')">
              <el-icon><Plus /></el-icon>
              <span style="margin-left:6px;">เพิ่มรายการ</span>
            </el-button>
          </div>

          <div v-loading="loadingBest" class="sortable-list" ref="bestDealsListRef">
            <div v-if="bestDeals.length === 0" class="empty-state">
              <el-empty description="ยังไม่มีรายการ กด 'เพิ่มรายการ' เพื่อเริ่มต้น" />
            </div>
            <div
              v-for="item in bestDeals"
              :key="item.id"
              class="sortable-item"
              :data-id="item.id"
            >
              <el-icon class="drag-handle"><Rank /></el-icon>
              <div class="item-order">{{ item.order }}</div>
              <el-tag
                size="small"
                :type="item.type === 'sippcode' ? 'warning' : 'primary'"
                class="type-tag"
              >
                {{ item.type === 'sippcode' ? 'SIPP' : 'รุ่น' }}
              </el-tag>
              <div class="item-info">
                <template v-if="item.type === 'sippcode'">
                  <span class="sipp-code">{{ item.sippcode }}</span>
                  <span class="sipp-category">{{ item.categoryName }}</span>
                </template>
                <template v-else>
                  <span class="brand-name">{{ item.brandName }}</span>
                  <span class="model-name">{{ item.modelName }}</span>
                </template>
              </div>
              <el-button
                type="danger"
                size="small"
                plain
                circle
                @click="removeItem('bestDeals', item.id)"
              >
                <el-icon><Delete /></el-icon>
              </el-button>
            </div>
          </div>
        </el-tab-pane>

        <!-- รุ่นรถยอดนิยม -->
        <el-tab-pane name="popularModels">
          <template #label>
            <span class="tab-label">
              <el-icon><Trophy /></el-icon>
              รุ่นรถยอดนิยม
            </span>
          </template>

          <div class="tab-description">
            เลือกรุ่นรถหรือ SIPP Code ที่จะแสดงในส่วน <strong>"รุ่นรถยอดนิยม"</strong> บนหน้าเว็บ และลากเรียงลำดับได้ตามต้องการ
          </div>

          <div class="list-toolbar">
            <span class="item-count">{{ popularModels.length }} รายการ</span>
            <el-button type="primary" @click="openAddDialog('popularModels')">
              <el-icon><Plus /></el-icon>
              <span style="margin-left:6px;">เพิ่มรายการ</span>
            </el-button>
          </div>

          <div v-loading="loadingPopular" class="sortable-list" ref="popularModelsListRef">
            <div v-if="popularModels.length === 0" class="empty-state">
              <el-empty description="ยังไม่มีรายการ กด 'เพิ่มรายการ' เพื่อเริ่มต้น" />
            </div>
            <div
              v-for="item in popularModels"
              :key="item.id"
              class="sortable-item"
              :data-id="item.id"
            >
              <el-icon class="drag-handle"><Rank /></el-icon>
              <div class="item-order">{{ item.order }}</div>
              <el-tag
                size="small"
                :type="item.type === 'sippcode' ? 'warning' : 'primary'"
                class="type-tag"
              >
                {{ item.type === 'sippcode' ? 'SIPP' : 'รุ่น' }}
              </el-tag>
              <div class="item-info">
                <template v-if="item.type === 'sippcode'">
                  <span class="sipp-code">{{ item.sippcode }}</span>
                  <span class="sipp-category">{{ item.categoryName }}</span>
                </template>
                <template v-else>
                  <span class="brand-name">{{ item.brandName }}</span>
                  <span class="model-name">{{ item.modelName }}</span>
                </template>
              </div>
              <el-button
                type="danger"
                size="small"
                plain
                circle
                @click="removeItem('popularModels', item.id)"
              >
                <el-icon><Delete /></el-icon>
              </el-button>
            </div>
          </div>
        </el-tab-pane>

      </el-tabs>
    </div>

    <!-- Bottom Save Bar -->
    <transition name="slide-up">
      <div v-if="hasUnsavedChanges" class="bottom-action-bar">
        <div class="action-bar-content">
          <div class="unsaved-notice">
            <el-icon><Warning /></el-icon>
            มีการเปลี่ยนแปลงที่ยังไม่ได้บันทึก
          </div>
          <div class="action-buttons">
            <el-button @click="cancelChanges">ยกเลิก</el-button>
            <el-button type="primary" :loading="saving" @click="saveChanges">บันทึก</el-button>
          </div>
        </div>
      </div>
    </transition>

    <!-- Add Dialog -->
    <el-dialog
      v-model="addDialogVisible"
      :title="addDialogTitle"
      width="520px"
      :close-on-click-modal="false"
    >
      <el-tabs v-model="dialogTab" class="dialog-tabs">

        <!-- Tab: เลือกด้วยรุ่นรถ -->
        <el-tab-pane label="รุ่นรถ" name="model">
          <div class="dialog-search">
            <el-input
              v-model="searchKeyword"
              placeholder="ค้นหายี่ห้อหรือรุ่น..."
              clearable
              prefix-icon="Search"
            />
          </div>

          <div class="brand-model-list">
            <template v-for="brand in filteredBrands" :key="brand.id">
              <div class="brand-group">
                <div class="brand-header">{{ brand.name }}</div>
                <div
                  v-for="model in brand.models"
                  :key="model.id"
                  class="select-row"
                  @click="selectModel(brand, model)"
                >
                  <span class="row-name">{{ model.name }}</span>
                  <el-icon class="row-add"><Plus /></el-icon>
                </div>
              </div>
            </template>
            <el-empty v-if="filteredBrands.length === 0" description="ไม่พบรุ่นรถที่ค้นหา" />
          </div>
        </el-tab-pane>

        <!-- Tab: เลือกด้วย SIPP Code -->
        <el-tab-pane name="sippcode">
          <template #label>
            <span>SIPP Code</span>
          </template>

          <div class="dialog-search">
            <el-input
              v-model="sippSearchKeyword"
              placeholder="ค้นหา SIPP Code หรือหมวดหมู่..."
              clearable
              prefix-icon="Search"
            />
          </div>

          <div class="brand-model-list">
            <template v-for="cat in filteredCategories" :key="cat.id">
              <div class="brand-group">
                <div class="brand-header">{{ cat.name }}</div>
                <div
                  v-for="code in cat.sippcodes"
                  :key="code"
                  class="select-row"
                  @click="selectSippcode(cat, code)"
                >
                  <div class="sipp-row-info">
                    <span class="sipp-row-code">{{ code }}</span>
                    <span class="sipp-row-category">{{ cat.name }}</span>
                  </div>
                  <el-icon class="row-add"><Plus /></el-icon>
                </div>
              </div>
            </template>
            <el-empty v-if="filteredCategories.length === 0" description="ไม่พบ SIPP Code ที่ค้นหา" />
          </div>
        </el-tab-pane>

      </el-tabs>

      <template #footer>
        <el-button @click="addDialogVisible = false">ปิด</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick, watch } from 'vue'
import { useApi } from '@/composables/useApi'
import Sortable from 'sortablejs'
import { ElMessage } from 'element-plus'

const {
  getBestDeals, updateBestDeals,
  getPopularModels, updatePopularModels,
  getCarBrands, getCarCategories
} = useApi()

const activeTab = ref('bestDeals')
const bestDeals = ref([])
const popularModels = ref([])
const allBrands = ref([])
const allCategories = ref([])
const loadingBest = ref(false)
const loadingPopular = ref(false)
const saving = ref(false)
const hasUnsavedChanges = ref(false)

let originalBestDeals = []
let originalPopularModels = []

const bestDealsListRef = ref(null)
const popularModelsListRef = ref(null)
let sortableBest = null
let sortablePopular = null

// Dialog state
const addDialogVisible = ref(false)
const addingTarget = ref('')
const dialogTab = ref('model')
const searchKeyword = ref('')
const sippSearchKeyword = ref('')

const addDialogTitle = computed(() =>
  addingTarget.value === 'bestDeals' ? 'เพิ่มรายการ — ดีลที่ดีที่สุด' : 'เพิ่มรายการ — รุ่นรถยอดนิยม'
)

const currentList = computed(() =>
  addingTarget.value === 'bestDeals' ? bestDeals.value : popularModels.value
)

// Filtered brands for model tab — excludes already-selected models
const filteredBrands = computed(() => {
  const kw = searchKeyword.value.toLowerCase()
  return allBrands.value
    .map(brand => ({
      ...brand,
      models: brand.models.filter(m => {
        if (isModelSelected(brand, m)) return false
        if (!kw) return true
        return m.name.toLowerCase().includes(kw) || brand.name.toLowerCase().includes(kw)
      })
    }))
    .filter(brand => brand.models.length > 0)
})

// Filtered categories for SIPP tab — excludes already-selected SIPP codes
const filteredCategories = computed(() => {
  const kw = sippSearchKeyword.value.toLowerCase()
  return allCategories.value
    .map(cat => ({
      ...cat,
      sippcodes: cat.sippcodes.filter(code => {
        if (isSippcodeSelected(code)) return false
        if (!kw) return true
        return code.toLowerCase().includes(kw) || cat.name.toLowerCase().includes(kw)
      })
    }))
    .filter(cat => cat.sippcodes.length > 0)
})

const isModelSelected = (brand, model) =>
  currentList.value.some(i => i.type === 'model' && i.brandId === brand.id && i.modelId === model.id)

const isSippcodeSelected = (code) =>
  currentList.value.some(i => i.type === 'sippcode' && i.sippcode === code)

const loadData = async () => {
  loadingBest.value = true
  loadingPopular.value = true
  const [best, popular, brands, cats] = await Promise.all([
    getBestDeals(), getPopularModels(), getCarBrands(), getCarCategories()
  ])
  bestDeals.value = best.map(i => ({ type: 'model', ...i }))
  popularModels.value = popular.map(i => ({ type: 'model', ...i }))
  allBrands.value = brands
  allCategories.value = cats
  originalBestDeals = JSON.parse(JSON.stringify(bestDeals.value))
  originalPopularModels = JSON.parse(JSON.stringify(popularModels.value))
  loadingBest.value = false
  loadingPopular.value = false
  await nextTick()
  initSortables()
}

const initSortables = () => {
  if (bestDealsListRef.value) {
    sortableBest?.destroy()
    sortableBest = Sortable.create(bestDealsListRef.value, {
      handle: '.drag-handle',
      animation: 150,
      onEnd: () => { reindexFromDOM('bestDeals'); hasUnsavedChanges.value = true }
    })
  }
  if (popularModelsListRef.value) {
    sortablePopular?.destroy()
    sortablePopular = Sortable.create(popularModelsListRef.value, {
      handle: '.drag-handle',
      animation: 150,
      onEnd: () => { reindexFromDOM('popularModels'); hasUnsavedChanges.value = true }
    })
  }
}

const reindexFromDOM = (target) => {
  const list = target === 'bestDeals' ? bestDeals : popularModels
  const el = target === 'bestDeals' ? bestDealsListRef.value : popularModelsListRef.value
  if (!el) return
  const ids = [...el.querySelectorAll('.sortable-item')].map(el => parseInt(el.dataset.id))
  list.value = ids.map((id, index) => ({
    ...list.value.find(i => i.id === id),
    order: index + 1
  }))
}

const removeItem = (target, id) => {
  const list = target === 'bestDeals' ? bestDeals : popularModels
  list.value = list.value.filter(i => i.id !== id).map((i, idx) => ({ ...i, order: idx + 1 }))
  hasUnsavedChanges.value = true
}

const openAddDialog = (target) => {
  addingTarget.value = target
  dialogTab.value = 'model'
  searchKeyword.value = ''
  sippSearchKeyword.value = ''
  addDialogVisible.value = true
}

const selectModel = (brand, model) => {
  const list = addingTarget.value === 'bestDeals' ? bestDeals : popularModels
  list.value.push({
    id: Date.now(),
    type: 'model',
    brandId: brand.id,
    brandName: brand.name,
    modelId: model.id,
    modelName: model.name,
    order: list.value.length + 1
  })
  hasUnsavedChanges.value = true
}

const selectSippcode = (category, code) => {
  const list = addingTarget.value === 'bestDeals' ? bestDeals : popularModels
  list.value.push({
    id: Date.now(),
    type: 'sippcode',
    sippcode: code,
    categoryId: category.id,
    categoryName: category.name,
    order: list.value.length + 1
  })
  hasUnsavedChanges.value = true
}

const saveChanges = async () => {
  saving.value = true
  await Promise.all([updateBestDeals(bestDeals.value), updatePopularModels(popularModels.value)])
  originalBestDeals = JSON.parse(JSON.stringify(bestDeals.value))
  originalPopularModels = JSON.parse(JSON.stringify(popularModels.value))
  hasUnsavedChanges.value = false
  saving.value = false
  ElMessage.success('บันทึกเรียบร้อยแล้ว')
}

const cancelChanges = () => {
  bestDeals.value = JSON.parse(JSON.stringify(originalBestDeals))
  popularModels.value = JSON.parse(JSON.stringify(originalPopularModels))
  hasUnsavedChanges.value = false
}

watch(activeTab, async () => {
  await nextTick()
  initSortables()
})

onMounted(loadData)
</script>

<style lang="scss" scoped>
.car-featured-sorting {
  padding-bottom: 80px;
}

.sticky-header {
  position: sticky;
  top: 0;
  z-index: 10;
  background: #f5f5f5;
  padding-bottom: 12px;

  .page-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 20px 0 0;

    h1 {
      margin: 0;
      font-size: 22px;
      font-weight: 700;
      color: #1a1a2e;
    }
  }
}

.content-card {
  background: #fff;
  border-radius: 12px;
  padding: 24px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.06);
}

.tab-label {
  display: flex;
  align-items: center;
  gap: 6px;
}

.tab-description {
  font-size: 13px;
  color: #888;
  margin: 8px 0 16px;
  padding: 10px 14px;
  background: #fafafa;
  border-radius: 8px;
  border-left: 3px solid #FF595A;
}

.list-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 12px;

  .item-count {
    font-size: 13px;
    color: #999;
  }
}

.sortable-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  min-height: 60px;
}

.sortable-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  background: #fafafa;
  border: 1px solid #f0f0f0;
  border-radius: 8px;
  transition: box-shadow 0.15s;

  &:hover {
    box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  }

  .drag-handle {
    cursor: grab;
    color: #ccc;
    font-size: 18px;
    &:active { cursor: grabbing; }
  }

  .item-order {
    width: 24px;
    height: 24px;
    background: #FF595A;
    color: #fff;
    border-radius: 50%;
    font-size: 12px;
    font-weight: 700;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }

  .type-tag {
    flex-shrink: 0;
  }

  .item-info {
    flex: 1;
    display: flex;
    align-items: center;
    gap: 8px;

    .brand-name {
      font-size: 13px;
      color: #999;
    }

    .model-name {
      font-size: 15px;
      font-weight: 600;
      color: #1a1a2e;
    }

    .sipp-code {
      font-size: 15px;
      font-weight: 700;
      color: #E6A23C;
      font-family: monospace;
      letter-spacing: 0.5px;
    }

    .sipp-category {
      font-size: 13px;
      color: #999;
    }
  }
}

.empty-state {
  padding: 20px 0;
}

// Dialog
.dialog-tabs {
  :deep(.el-tabs__header) {
    margin-bottom: 12px;
  }
}

.dialog-search {
  margin-bottom: 12px;
}

.brand-model-list {
  max-height: 360px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 12px;

  .brand-group {
    .brand-header {
      font-size: 11px;
      font-weight: 700;
      color: #aaa;
      text-transform: uppercase;
      letter-spacing: 0.5px;
      padding: 4px 0 6px;
      border-bottom: 1px solid #f0f0f0;
      margin-bottom: 4px;
    }

    .select-row {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 8px 10px;
      border-radius: 6px;
      cursor: pointer;
      transition: background 0.15s;

      &:hover:not(.is-disabled) {
        background: #FFF5F5;
      }

      .row-name {
        font-size: 14px;
        color: #333;
      }

      .row-add {
        color: #FF595A;
        font-size: 16px;
      }

      .sipp-row-info {
        display: flex;
        align-items: center;
        gap: 10px;

        .sipp-row-code {
          font-size: 14px;
          font-weight: 700;
          color: #E6A23C;
          font-family: monospace;
          letter-spacing: 0.5px;
          min-width: 60px;
        }

        .sipp-row-category {
          font-size: 13px;
          color: #888;
        }
      }
    }
  }
}

// Bottom save bar
.bottom-action-bar {
  position: fixed;
  bottom: 0;
  left: 220px;
  right: 0;
  background: #fff;
  border-top: 1px solid #f0f0f0;
  box-shadow: 0 -4px 12px rgba(0,0,0,0.06);
  z-index: 100;
  padding: 12px 24px;

  .action-bar-content {
    display: flex;
    align-items: center;
    justify-content: space-between;
    max-width: 1200px;
  }

  .unsaved-notice {
    display: flex;
    align-items: center;
    gap: 8px;
    color: #E6A23C;
    font-size: 14px;
    font-weight: 500;
  }

  .action-buttons {
    display: flex;
    gap: 8px;
  }
}

.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.25s ease, opacity 0.25s ease;
}
.slide-up-enter-from,
.slide-up-leave-to {
  transform: translateY(100%);
  opacity: 0;
}
</style>
