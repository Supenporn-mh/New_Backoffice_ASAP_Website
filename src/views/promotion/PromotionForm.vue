<template>
  <div class="promotion-form">
    <div class="page-header">
      <h1>{{ isEdit ? 'แก้ไขโปรโมชัน' : 'เพิ่มโปรโมชัน' }}</h1>
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

        <el-form-item label="ชื่อโปรโมชัน (สำหรับ Admin)" prop="title">
          <el-input
            v-model="form.title"
            placeholder="กรอกชื่อโปรโมชัน"
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
          <el-col :span="8">
            <el-form-item label="วันเริ่มต้น" prop="startDate">
              <el-date-picker
                v-model="form.startDate"
                type="date"
                placeholder="เลือกวันเริ่มต้น"
                format="DD/MM/YYYY"
                value-format="YYYY-MM-DD"
                style="width: 100%;"
              />
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="วันสิ้นสุด" prop="endDate">
              <el-date-picker
                v-model="form.endDate"
                type="date"
                placeholder="เลือกวันสิ้นสุด"
                format="DD/MM/YYYY"
                value-format="YYYY-MM-DD"
                style="width: 100%;"
              />
            </el-form-item>
          </el-col>
          <el-col :span="8">
            <el-form-item label="ลำดับการแสดง" prop="order">
              <el-input-number
                v-model="form.order"
                :min="1"
                :max="3"
                style="width: 100%;"
              />
            </el-form-item>
          </el-col>
        </el-row>

        <el-form-item label="สถานะ" prop="isActive">
          <el-switch
            v-model="form.isActive"
            inline-prompt
            active-text="แสดง"
            inactive-text="ซ่อน"
          />
        </el-form-item>
      </div>

      <div class="form-section">
        <div class="form-section-title">รูปภาพโปรโมชัน (Desktop) <span class="size-hint">แนะนำขนาด 736 x 507 px</span></div>
        <el-tabs type="card" class="lang-tabs">
          <el-tab-pane label="🇹🇭 ไทย">
            <div class="image-upload-wrapper">
              <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false" accept="image/*"
                @change="(f) => handleImageChange(f, 'th')">
                <div v-if="form.image.th" class="image-preview">
                  <img :src="form.image.th" />
                  <div class="image-overlay"><el-icon><Plus /></el-icon><span>เปลี่ยนรูป</span></div>
                </div>
                <div v-else class="upload-placeholder">
                  <el-icon><Plus /></el-icon><span>อัปโหลดรูปภาพ</span>
                </div>
              </el-upload>
              <el-input v-model="form.imageAlt.th" placeholder="Alt Text (ภาษาไทย)" class="mt-2" />
            </div>
          </el-tab-pane>
          <el-tab-pane label="🇬🇧 EN">
            <div class="image-upload-wrapper">
              <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false" accept="image/*"
                @change="(f) => handleImageChange(f, 'en')">
                <div v-if="form.image.en" class="image-preview">
                  <img :src="form.image.en" />
                  <div class="image-overlay"><el-icon><Plus /></el-icon><span>เปลี่ยนรูป</span></div>
                </div>
                <div v-else class="upload-placeholder">
                  <el-icon><Plus /></el-icon><span>Upload Image</span>
                </div>
              </el-upload>
              <el-input v-model="form.imageAlt.en" placeholder="Alt Text (English)" class="mt-2" />
            </div>
          </el-tab-pane>
          <el-tab-pane label="🇨🇳 中文">
            <div class="image-upload-wrapper">
              <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false" accept="image/*"
                @change="(f) => handleImageChange(f, 'zh')">
                <div v-if="form.image.zh" class="image-preview">
                  <img :src="form.image.zh" />
                  <div class="image-overlay"><el-icon><Plus /></el-icon><span>更换图片</span></div>
                </div>
                <div v-else class="upload-placeholder">
                  <el-icon><Plus /></el-icon><span>上传图片</span>
                </div>
              </el-upload>
              <el-input v-model="form.imageAlt.zh" placeholder="Alt Text (中文)" class="mt-2" />
            </div>
          </el-tab-pane>
        </el-tabs>
        <div v-if="imageError" class="field-error">{{ imageError }}</div>
      </div>

      <div class="form-section">
        <div class="form-section-title">รูปภาพโปรโมชัน (Mobile) <span class="size-hint">แนะนำขนาด 390 x 270 px</span></div>
        <el-tabs type="card" class="lang-tabs">
          <el-tab-pane label="🇹🇭 ไทย">
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
          <el-tab-pane label="🇬🇧 EN">
            <div class="image-upload-wrapper">
              <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false" accept="image/*"
                @change="(f) => handleMobileImageChange(f, 'en')">
                <div v-if="form.imageMobile.en" class="image-preview image-preview--mobile">
                  <img :src="form.imageMobile.en" />
                  <div class="image-overlay"><el-icon><Plus /></el-icon><span>Change Image</span></div>
                </div>
                <div v-else class="upload-placeholder upload-placeholder--mobile">
                  <el-icon><Plus /></el-icon><span>Upload Image</span>
                </div>
              </el-upload>
              <el-input v-model="form.imageAltMobile.en" placeholder="Alt Text (English)" class="mt-2" />
            </div>
          </el-tab-pane>
          <el-tab-pane label="🇨🇳 中文">
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
          {{ isEdit ? 'บันทึกการแก้ไข' : 'สร้างโปรโมชัน' }}
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
const { getPromotion, createPromotion, updatePromotion } = useApi()

const formRef = ref()
const saving = ref(false)
const isDirty = ref(false)
const imageError = ref('')
const mobileImageError = ref('')

const isEdit = computed(() => !!route.params.id)

const form = reactive({
  title: '',
  image: { th: '', en: '', zh: '' },
  imageAlt: { th: '', en: '', zh: '' },
  imageMobile: { th: '', en: '', zh: '' },
  imageAltMobile: { th: '', en: '', zh: '' },
  link: '',
  startDate: '',
  endDate: '',
  order: 1,
  isActive: true
})

watch(() => form, () => { isDirty.value = true }, { deep: true })

const validateEndDate = (rule, value, callback) => {
  if (value && form.startDate && value < form.startDate) {
    callback(new Error('วันสิ้นสุดต้องมากกว่าวันเริ่มต้น'))
  } else {
    callback()
  }
}

const rules = {
  title: [
    { required: true, message: 'กรุณากรอกชื่อโปรโมชัน', trigger: 'blur' }
  ],
  startDate: [
    { required: true, message: 'กรุณาเลือกวันเริ่มต้น', trigger: 'change' }
  ],
  endDate: [
    { required: true, message: 'กรุณาเลือกวันสิ้นสุด', trigger: 'change' },
    { validator: validateEndDate, trigger: 'change' }
  ],
  order: [
    { required: true, message: 'กรุณากรอกลำดับ', trigger: 'blur' }
  ]
}

const handleImageChange = (file, lang) => {
  const reader = new FileReader()
  reader.onload = (e) => {
    form.image[lang] = e.target.result
    if (lang === 'th') imageError.value = ''
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

  imageError.value = ''

  await formRef.value.validate(async (valid) => {
    if (valid) {
      if (!form.image.th) {
        imageError.value = 'กรุณาอัปโหลดรูปภาพโปรโมชัน (ภาษาไทย)'
        return
      }

      saving.value = true
      try {
        if (isEdit.value) {
          await updatePromotion(route.params.id, form)
          ElMessage.success('แก้ไขโปรโมชันสำเร็จ')
        } else {
          await createPromotion(form)
          ElMessage.success('สร้างโปรโมชันสำเร็จ')
        }
        isDirty.value = false
        router.push('/promotions')
      } catch (error) {
        ElMessage.error('เกิดข้อผิดพลาด กรุณาลองใหม่')
      } finally {
        saving.value = false
      }
    }
  })
}

const loadPromotion = async () => {
  if (isEdit.value) {
    try {
      const data = await getPromotion(route.params.id)
      if (data) {
        if (data.image && typeof data.image === 'object') Object.assign(form.image, data.image)
        if (data.imageAlt && typeof data.imageAlt === 'object') Object.assign(form.imageAlt, data.imageAlt)
        if (data.imageMobile && typeof data.imageMobile === 'object') Object.assign(form.imageMobile, data.imageMobile)
        if (data.imageAltMobile && typeof data.imageAltMobile === 'object') Object.assign(form.imageAltMobile, data.imageAltMobile)
        form.title = data.title || ''
        form.link = data.link || ''
        form.startDate = data.startDate || ''
        form.endDate = data.endDate || ''
        form.order = data.order || 1
        form.isActive = data.isActive ?? true
        nextTick(() => { isDirty.value = false })
      } else {
        ElMessage.error('ไม่พบข้อมูลโปรโมชัน')
        router.push('/promotions')
      }
    } catch (error) {
      ElMessage.error('ไม่สามารถโหลดข้อมูลได้')
      router.push('/promotions')
    }
  }
}

onMounted(() => {
  loadPromotion()
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
  max-width: 400px;
}

.image-uploader {
  :deep(.el-upload) {
    width: 100%;
  }
}

.image-preview {
  width: 100%;
  height: 280px;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
  border: 1px solid #ddd;

  &--mobile {
    height: 200px;
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
  height: 280px;
  border: 2px dashed #ddd;

  &--mobile {
    height: 200px;
  }
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #999;
  cursor: pointer;
  transition: border-color 0.2s;

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
