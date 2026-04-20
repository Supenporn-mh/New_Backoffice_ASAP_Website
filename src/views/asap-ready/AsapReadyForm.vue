<template>
  <div class="asap-ready-form">
    <div class="page-header">
      <h1>{{ isEdit ? 'แก้ไขรูปภาพ ASAP พร้อมให้บริการ' : 'เพิ่มรูปภาพ ASAP พร้อมให้บริการ' }}</h1>
    </div>

    <el-form
      ref="formRef"
      :model="form"
      :rules="rules"
      label-position="top"
      class="content-card"
    >
      <!-- ข้อมูลทั่วไป -->
      <div class="form-section">
        <div class="form-section-title">ข้อมูลทั่วไป</div>

        <el-form-item label="ชื่อ (สำหรับ Admin)" prop="title">
          <el-input
            v-model="form.title"
            placeholder="กรอกชื่อรายการ เพื่อใช้อ้างอิงใน Admin"
            maxlength="100"
            show-word-limit
          />
        </el-form-item>

        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="ลำดับการแสดง" prop="order">
              <el-input-number v-model="form.order" :min="1" :max="99" style="width: 100%;" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="สถานะ" prop="isActive">
              <el-switch
                v-model="form.isActive"
                inline-prompt
                active-text="แสดง"
                inactive-text="ซ่อน"
              />
            </el-form-item>
          </el-col>
        </el-row>
      </div>

      <!-- รูปภาพ Desktop -->
      <div class="form-section">
        <div class="form-section-title">
          รูปภาพ Desktop
          <span class="size-hint">แนะนำขนาด 420 x 470 px</span>
        </div>
        <el-tabs type="card" class="lang-tabs">
          <el-tab-pane label="🇹🇭 ไทย">
            <div class="image-upload-wrapper">
              <el-upload
                class="image-uploader"
                :show-file-list="false"
                :auto-upload="false"
                accept="image/*"
                @change="(f) => handleDesktopImageChange(f, 'th')"
              >
                <div v-if="form.imageDesktop.th" class="image-preview">
                  <img :src="form.imageDesktop.th" />
                  <div class="image-overlay">
                    <el-icon><Plus /></el-icon><span>เปลี่ยนรูป</span>
                  </div>
                </div>
                <div v-else class="upload-placeholder">
                  <el-icon><Plus /></el-icon><span>อัปโหลดรูปภาพ</span>
                </div>
              </el-upload>
              <el-button
                v-if="form.imageDesktop.th"
                type="danger"
                plain
                size="small"
                class="remove-btn"
                @click="form.imageDesktop.th = ''"
              >
                <el-icon><Delete /></el-icon> ลบรูปภาพ
              </el-button>
              <el-input v-model="form.imageAltDesktop.th" placeholder="Alt Text (ภาษาไทย)" class="mt-2" />
            </div>
          </el-tab-pane>

          <el-tab-pane label="🇬🇧 EN">
            <div class="image-upload-wrapper">
              <el-upload
                class="image-uploader"
                :show-file-list="false"
                :auto-upload="false"
                accept="image/*"
                @change="(f) => handleDesktopImageChange(f, 'en')"
              >
                <div v-if="form.imageDesktop.en" class="image-preview">
                  <img :src="form.imageDesktop.en" />
                  <div class="image-overlay">
                    <el-icon><Plus /></el-icon><span>Change Image</span>
                  </div>
                </div>
                <div v-else class="upload-placeholder">
                  <el-icon><Plus /></el-icon><span>Upload Image</span>
                </div>
              </el-upload>
              <el-button
                v-if="form.imageDesktop.en"
                type="danger"
                plain
                size="small"
                class="remove-btn"
                @click="form.imageDesktop.en = ''"
              >
                <el-icon><Delete /></el-icon> Remove
              </el-button>
              <el-input v-model="form.imageAltDesktop.en" placeholder="Alt Text (English)" class="mt-2" />
            </div>
          </el-tab-pane>

          <el-tab-pane label="🇨🇳 中文">
            <div class="image-upload-wrapper">
              <el-upload
                class="image-uploader"
                :show-file-list="false"
                :auto-upload="false"
                accept="image/*"
                @change="(f) => handleDesktopImageChange(f, 'zh')"
              >
                <div v-if="form.imageDesktop.zh" class="image-preview">
                  <img :src="form.imageDesktop.zh" />
                  <div class="image-overlay">
                    <el-icon><Plus /></el-icon><span>更换图片</span>
                  </div>
                </div>
                <div v-else class="upload-placeholder">
                  <el-icon><Plus /></el-icon><span>上传图片</span>
                </div>
              </el-upload>
              <el-button
                v-if="form.imageDesktop.zh"
                type="danger"
                plain
                size="small"
                class="remove-btn"
                @click="form.imageDesktop.zh = ''"
              >
                <el-icon><Delete /></el-icon> 删除
              </el-button>
              <el-input v-model="form.imageAltDesktop.zh" placeholder="Alt Text (中文)" class="mt-2" />
            </div>
          </el-tab-pane>
        </el-tabs>
        <div v-if="desktopImageError" class="field-error">{{ desktopImageError }}</div>
      </div>

      <!-- รูปภาพ Mobile -->
      <div class="form-section">
        <div class="form-section-title">
          รูปภาพ Mobile
          <span class="size-hint">แนะนำขนาด 300 x 335 px</span>
        </div>
        <el-tabs type="card" class="lang-tabs">
          <el-tab-pane label="🇹🇭 ไทย">
            <div class="image-upload-wrapper">
              <el-upload
                class="image-uploader"
                :show-file-list="false"
                :auto-upload="false"
                accept="image/*"
                @change="(f) => handleMobileImageChange(f, 'th')"
              >
                <div v-if="form.imageMobile.th" class="image-preview image-preview--mobile">
                  <img :src="form.imageMobile.th" />
                  <div class="image-overlay">
                    <el-icon><Plus /></el-icon><span>เปลี่ยนรูป</span>
                  </div>
                </div>
                <div v-else class="upload-placeholder upload-placeholder--mobile">
                  <el-icon><Plus /></el-icon><span>อัปโหลดรูปภาพ</span>
                </div>
              </el-upload>
              <el-button
                v-if="form.imageMobile.th"
                type="danger"
                plain
                size="small"
                class="remove-btn"
                @click="form.imageMobile.th = ''"
              >
                <el-icon><Delete /></el-icon> ลบรูปภาพ
              </el-button>
              <el-input v-model="form.imageAltMobile.th" placeholder="Alt Text (ภาษาไทย)" class="mt-2" />
            </div>
          </el-tab-pane>

          <el-tab-pane label="🇬🇧 EN">
            <div class="image-upload-wrapper">
              <el-upload
                class="image-uploader"
                :show-file-list="false"
                :auto-upload="false"
                accept="image/*"
                @change="(f) => handleMobileImageChange(f, 'en')"
              >
                <div v-if="form.imageMobile.en" class="image-preview image-preview--mobile">
                  <img :src="form.imageMobile.en" />
                  <div class="image-overlay">
                    <el-icon><Plus /></el-icon><span>Change Image</span>
                  </div>
                </div>
                <div v-else class="upload-placeholder upload-placeholder--mobile">
                  <el-icon><Plus /></el-icon><span>Upload Image</span>
                </div>
              </el-upload>
              <el-button
                v-if="form.imageMobile.en"
                type="danger"
                plain
                size="small"
                class="remove-btn"
                @click="form.imageMobile.en = ''"
              >
                <el-icon><Delete /></el-icon> Remove
              </el-button>
              <el-input v-model="form.imageAltMobile.en" placeholder="Alt Text (English)" class="mt-2" />
            </div>
          </el-tab-pane>

          <el-tab-pane label="🇨🇳 中文">
            <div class="image-upload-wrapper">
              <el-upload
                class="image-uploader"
                :show-file-list="false"
                :auto-upload="false"
                accept="image/*"
                @change="(f) => handleMobileImageChange(f, 'zh')"
              >
                <div v-if="form.imageMobile.zh" class="image-preview image-preview--mobile">
                  <img :src="form.imageMobile.zh" />
                  <div class="image-overlay">
                    <el-icon><Plus /></el-icon><span>更换图片</span>
                  </div>
                </div>
                <div v-else class="upload-placeholder upload-placeholder--mobile">
                  <el-icon><Plus /></el-icon><span>上传图片</span>
                </div>
              </el-upload>
              <el-button
                v-if="form.imageMobile.zh"
                type="danger"
                plain
                size="small"
                class="remove-btn"
                @click="form.imageMobile.zh = ''"
              >
                <el-icon><Delete /></el-icon> 删除
              </el-button>
              <el-input v-model="form.imageAltMobile.zh" placeholder="Alt Text (中文)" class="mt-2" />
            </div>
          </el-tab-pane>
        </el-tabs>
        <div v-if="mobileImageError" class="field-error">{{ mobileImageError }}</div>
      </div>

      <div class="form-actions">
        <el-button @click="$router.back()">ยกเลิก</el-button>
        <el-button type="primary" :loading="saving" :disabled="!isDirty || saving" @click="handleSubmit">
          {{ isEdit ? 'บันทึกการแก้ไข' : 'บันทึก' }}
        </el-button>
      </div>
    </el-form>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, watch, nextTick } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { Plus, Delete } from '@element-plus/icons-vue'
import { useApi } from '@/composables/useApi'

const route = useRoute()
const router = useRouter()
const { getAsapReadyItem, createAsapReadyItem, updateAsapReadyItem } = useApi()

const formRef = ref()
const saving = ref(false)
const isDirty = ref(false)
const desktopImageError = ref('')
const mobileImageError = ref('')

const isEdit = computed(() => !!route.params.id)

const form = reactive({
  title: '',
  imageDesktop: { th: '', en: '', zh: '' },
  imageAltDesktop: { th: '', en: '', zh: '' },
  imageMobile: { th: '', en: '', zh: '' },
  imageAltMobile: { th: '', en: '', zh: '' },
  order: 1,
  isActive: true
})

watch(() => form, () => { isDirty.value = true }, { deep: true })

const rules = {
  title: [{ required: true, message: 'กรุณากรอกชื่อรายการ', trigger: 'blur' }],
  order: [{ required: true, message: 'กรุณากรอกลำดับ', trigger: 'blur' }]
}

const handleDesktopImageChange = (file, lang) => {
  const reader = new FileReader()
  reader.onload = (e) => {
    form.imageDesktop[lang] = e.target.result
    if (lang === 'th') desktopImageError.value = ''
  }
  reader.readAsDataURL(file.raw)
}

const handleMobileImageChange = (file, lang) => {
  const reader = new FileReader()
  reader.onload = (e) => {
    form.imageMobile[lang] = e.target.result
    if (lang === 'th') mobileImageError.value = ''
  }
  reader.readAsDataURL(file.raw)
}

const handleSubmit = async () => {
  if (!formRef.value) return

  desktopImageError.value = ''
  mobileImageError.value = ''

  await formRef.value.validate(async (valid) => {
    if (!valid) return

    if (!form.imageDesktop.th) {
      desktopImageError.value = 'กรุณาอัปโหลดรูปภาพ Desktop ภาษาไทย'
      return
    }
    if (!form.imageMobile.th) {
      mobileImageError.value = 'กรุณาอัปโหลดรูปภาพ Mobile ภาษาไทย'
      return
    }

    saving.value = true
    try {
      if (isEdit.value) {
        await updateAsapReadyItem(route.params.id, form)
        ElMessage.success('แก้ไขสำเร็จ')
      } else {
        await createAsapReadyItem(form)
        ElMessage.success('เพิ่มรูปภาพสำเร็จ')
      }
      isDirty.value = false
      router.push('/asap-ready')
    } catch {
      ElMessage.error('เกิดข้อผิดพลาด กรุณาลองใหม่')
    } finally {
      saving.value = false
    }
  })
}

const loadItem = async () => {
  if (!isEdit.value) return
  try {
    const data = await getAsapReadyItem(route.params.id)
    if (!data) {
      ElMessage.error('ไม่พบข้อมูล')
      router.push('/asap-ready')
      return
    }
    if (data.imageDesktop && typeof data.imageDesktop === 'object') {
      Object.assign(form.imageDesktop, data.imageDesktop)
    }
    if (data.imageAltDesktop && typeof data.imageAltDesktop === 'object') {
      Object.assign(form.imageAltDesktop, data.imageAltDesktop)
    }
    if (data.imageMobile && typeof data.imageMobile === 'object') {
      Object.assign(form.imageMobile, data.imageMobile)
    }
    if (data.imageAltMobile && typeof data.imageAltMobile === 'object') {
      Object.assign(form.imageAltMobile, data.imageAltMobile)
    }
    form.title = data.title || ''
    form.order = data.order || 1
    form.isActive = data.isActive ?? true
    nextTick(() => { isDirty.value = false })
  } catch {
    ElMessage.error('ไม่สามารถโหลดข้อมูลได้')
    router.push('/asap-ready')
  }
}

onMounted(() => loadItem())
</script>

<style lang="scss" scoped>
.form-section {
  margin-bottom: 32px;

  &-title {
    font-size: 16px;
    font-weight: 600;
    margin-bottom: 20px;
    padding-bottom: 12px;
    border-bottom: 1px solid #eee;
  }
}

.size-hint {
  font-size: 12px;
  font-weight: 400;
  color: #999;
  margin-left: 8px;
}

.lang-tabs {
  margin-bottom: 8px;

  :deep(.el-tabs__content) {
    padding: 16px 0 0;
  }
}

.image-upload-wrapper {
  width: 100%;
  max-width: 380px;
}

.image-uploader {
  :deep(.el-upload) {
    width: 100%;
  }
}

.image-preview {
  width: 100%;
  height: 425px;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
  border: 1px solid #ddd;

  &--mobile {
    height: 335px;
    max-width: 300px;
  }

  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .image-overlay {
    position: absolute;
    inset: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: #fff;
    opacity: 0;
    transition: opacity 0.2s;
    cursor: pointer;

    .el-icon {
      font-size: 24px;
      margin-bottom: 4px;
    }
  }

  &:hover .image-overlay {
    opacity: 1;
  }
}

.upload-placeholder {
  width: 100%;
  height: 425px;
  border: 2px dashed #ddd;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #999;
  cursor: pointer;
  transition: border-color 0.2s;

  &--mobile {
    height: 335px;
    max-width: 300px;
  }

  &:hover {
    border-color: #FF595A;
    color: #FF595A;
  }

  .el-icon {
    font-size: 32px;
    margin-bottom: 8px;
  }
}

.remove-btn {
  margin-top: 8px;
}

.field-error {
  font-size: 12px;
  color: #f56c6c;
  margin-top: 4px;
}

.mt-2 {
  margin-top: 8px;
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  padding-top: 24px;
  border-top: 1px solid #eee;
}
</style>
