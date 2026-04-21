<template>
  <div class="banner-form">
    <div class="page-header">
      <h1>{{ isEdit ? 'แก้ไขแบนเนอร์' : 'เพิ่มแบนเนอร์' }}</h1>
    </div>

    <el-form
      ref="formRef"
      :model="form"
      :rules="rules"
      label-position="top"
      class="content-card"
    >
      <div class="form-section">
        <div class="form-section-title">ข้อมูลทั่วไป</div>

        <el-form-item label="ชื่อแบนเนอร์ (สำหรับ Admin)" prop="title">
          <el-input
            v-model="form.title"
            placeholder="กรอกชื่อแบนเนอร์"
            maxlength="100"
            show-word-limit
          />
        </el-form-item>

        <el-form-item label="ลิงก์ URL (ถ้ามี)" prop="link">
          <el-input
            v-model="form.link"
            placeholder="https://example.com"
          />
        </el-form-item>

        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="ลำดับการแสดง" prop="order">
              <el-input-number
                v-model="form.order"
                :min="1"
                :max="5"
                style="width: 100%;"
              />
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

      <div class="form-section">
        <div class="form-section-title">รูปภาพ Desktop <span class="size-hint">แนะนำขนาด 1280 x 500 px</span></div>
        <el-tabs type="card" class="lang-tabs">
          <el-tab-pane label="TH ไทย">
            <div class="image-upload-wrapper">
              <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false" accept="image/*"
                @change="(f) => handleDesktopImageChange(f, 'th')">
                <div v-if="form.imageDesktop.th" class="image-preview">
                  <img :src="form.imageDesktop.th" />
                  <div class="image-overlay"><el-icon><Plus /></el-icon><span>เปลี่ยนรูป</span></div>
                </div>
                <div v-else class="upload-placeholder">
                  <el-icon><Plus /></el-icon><span>อัปโหลดรูปภาพ</span>
                </div>
              </el-upload>
              <el-input v-model="form.imageAltDesktop.th" placeholder="Alt Text (ภาษาไทย)" class="mt-2" />
            </div>
          </el-tab-pane>
          <el-tab-pane label="EN อังกฤษ">
            <div class="image-upload-wrapper">
              <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false" accept="image/*"
                @change="(f) => handleDesktopImageChange(f, 'en')">
                <div v-if="form.imageDesktop.en" class="image-preview">
                  <img :src="form.imageDesktop.en" />
                  <div class="image-overlay"><el-icon><Plus /></el-icon><span>เปลี่ยนรูป</span></div>
                </div>
                <div v-else class="upload-placeholder">
                  <el-icon><Plus /></el-icon><span>Upload Image</span>
                </div>
              </el-upload>
              <el-input v-model="form.imageAltDesktop.en" placeholder="Alt Text (English)" class="mt-2" />
            </div>
          </el-tab-pane>
          <el-tab-pane label="CN จีน">
            <div class="image-upload-wrapper">
              <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false" accept="image/*"
                @change="(f) => handleDesktopImageChange(f, 'zh')">
                <div v-if="form.imageDesktop.zh" class="image-preview">
                  <img :src="form.imageDesktop.zh" />
                  <div class="image-overlay"><el-icon><Plus /></el-icon><span>更换图片</span></div>
                </div>
                <div v-else class="upload-placeholder">
                  <el-icon><Plus /></el-icon><span>上传图片</span>
                </div>
              </el-upload>
              <el-input v-model="form.imageAltDesktop.zh" placeholder="Alt Text (中文)" class="mt-2" />
            </div>
          </el-tab-pane>
        </el-tabs>
        <div v-if="desktopImageError" class="field-error">{{ desktopImageError }}</div>
      </div>

      <div class="form-section">
        <div class="form-section-title">รูปภาพ Mobile <span class="size-hint">แนะนำขนาด 390 x 300 px</span></div>
        <el-tabs type="card" class="lang-tabs">
          <el-tab-pane label="TH ไทย">
            <div class="image-upload-wrapper">
              <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false" accept="image/*"
                @change="(f) => handleMobileImageChange(f, 'th')">
                <div v-if="form.imageMobile.th" class="image-preview image-preview--mobile">
                  <img :src="form.imageMobile.th" />
                  <div class="image-overlay"><el-icon><Plus /></el-icon><span>เปลี่ยนรูป</span></div>
                </div>
                <div v-else class="upload-placeholder upload-placeholder--mobile">
                  <el-icon><Plus /></el-icon><span>อัปโหลดรูปภาพ</span>
                </div>
              </el-upload>
              <el-input v-model="form.imageAltMobile.th" placeholder="Alt Text (ภาษาไทย)" class="mt-2" />
            </div>
          </el-tab-pane>
          <el-tab-pane label="EN อังกฤษ">
            <div class="image-upload-wrapper">
              <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false" accept="image/*"
                @change="(f) => handleMobileImageChange(f, 'en')">
                <div v-if="form.imageMobile.en" class="image-preview image-preview--mobile">
                  <img :src="form.imageMobile.en" />
                  <div class="image-overlay"><el-icon><Plus /></el-icon><span>เปลี่ยนรูป</span></div>
                </div>
                <div v-else class="upload-placeholder upload-placeholder--mobile">
                  <el-icon><Plus /></el-icon><span>Upload Image</span>
                </div>
              </el-upload>
              <el-input v-model="form.imageAltMobile.en" placeholder="Alt Text (English)" class="mt-2" />
            </div>
          </el-tab-pane>
          <el-tab-pane label="CN จีน">
            <div class="image-upload-wrapper">
              <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false" accept="image/*"
                @change="(f) => handleMobileImageChange(f, 'zh')">
                <div v-if="form.imageMobile.zh" class="image-preview image-preview--mobile">
                  <img :src="form.imageMobile.zh" />
                  <div class="image-overlay"><el-icon><Plus /></el-icon><span>更换图片</span></div>
                </div>
                <div v-else class="upload-placeholder upload-placeholder--mobile">
                  <el-icon><Plus /></el-icon><span>上传图片</span>
                </div>
              </el-upload>
              <el-input v-model="form.imageAltMobile.zh" placeholder="Alt Text (中文)" class="mt-2" />
            </div>
          </el-tab-pane>
        </el-tabs>
        <div v-if="mobileImageError" class="field-error">{{ mobileImageError }}</div>
      </div>

      <div class="form-actions">
        <el-button @click="$router.back()">ยกเลิก</el-button>
        <el-button type="primary" :loading="saving" :disabled="!isDirty || saving" @click="handleSubmit">
          {{ isEdit ? 'บันทึกการแก้ไข' : 'สร้างแบนเนอร์' }}
        </el-button>
      </div>
    </el-form>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, watch, nextTick } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { useApi } from '@/composables/useApi'

const route = useRoute()
const router = useRouter()
const { getBanner, getBanners, createBanner, updateBanner } = useApi()

const formRef = ref()
const saving = ref(false)
const isDirty = ref(false)
const desktopImageError = ref('')
const mobileImageError = ref('')
const existingOrders = ref([])

const isEdit = computed(() => !!route.params.id)

const form = reactive({
  title: '',
  imageDesktop: { th: '', en: '', zh: '' },
  imageAltDesktop: { th: '', en: '', zh: '' },
  imageMobile: { th: '', en: '', zh: '' },
  imageAltMobile: { th: '', en: '', zh: '' },
  link: '',
  order: 1,
  isActive: true
})

watch(() => form, () => { isDirty.value = true }, { deep: true })

const rules = {
  title: [
    { required: true, message: 'กรุณากรอกชื่อแบนเนอร์', trigger: 'blur' }
  ],
  order: [
    { required: true, message: 'กรุณากรอกลำดับ', trigger: 'blur' },
    {
      validator: (rule, value, callback) => {
        if (existingOrders.value.includes(value)) {
          callback(new Error('ลำดับนี้มีการใช้งานแล้ว กรุณาเลือกลำดับอื่น'))
        } else {
          callback()
        }
      },
      trigger: 'change'
    }
  ]
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
    if (valid) {
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
          await updateBanner(route.params.id, form)
          ElMessage.success('แก้ไขแบนเนอร์สำเร็จ')
        } else {
          await createBanner(form)
          ElMessage.success('สร้างแบนเนอร์สำเร็จ')
        }
        isDirty.value = false
        router.push('/banners')
      } catch (error) {
        ElMessage.error('เกิดข้อผิดพลาด กรุณาลองใหม่')
      } finally {
        saving.value = false
      }
    }
  })
}

const loadBanner = async () => {
  try {
    const list = await getBanners()
    if (isEdit.value) {
      existingOrders.value = list.filter(b => String(b.id) !== String(route.params.id)).map(b => b.order)
    } else {
      existingOrders.value = list.map(b => b.order)
      form.order = list.length + 1
    }
  } catch {}

  if (isEdit.value) {
    try {
      const data = await getBanner(route.params.id)
      if (data) {
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
        form.link = data.link || ''
        form.order = data.order || 1
        form.isActive = data.isActive ?? true
        nextTick(() => { isDirty.value = false })
      } else {
        ElMessage.error('ไม่พบข้อมูลแบนเนอร์')
        router.push('/banners')
      }
    } catch (error) {
      ElMessage.error('ไม่สามารถโหลดข้อมูลได้')
      router.push('/banners')
    }
  }
}

onMounted(() => {
  loadBanner()
})
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
}

.image-uploader {
  :deep(.el-upload) {
    width: 100%;
  }
}

.image-preview {
  width: 100%;
  height: 180px;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
  border: 1px solid #ddd;

  &--mobile {
    height: 220px;
    max-width: 200px;
  }

  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .image-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
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
  height: 180px;
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
    height: 220px;
    max-width: 200px;
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
