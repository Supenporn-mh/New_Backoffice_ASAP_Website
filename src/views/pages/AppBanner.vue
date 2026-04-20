<template>
  <div class="app-banner-page">
    <div class="page-header">
      <h1>จัดการ Home Application Banner</h1>
    </div>

    <div class="content-card">
      <el-form
        ref="formRef"
        :model="form"
        :rules="rules"
        label-position="top"
        v-loading="loading"
      >

        <!-- ===== PREVIEWS ===== -->
        <div class="previews-wrapper">

          <!-- Desktop Preview -->
          <div class="preview-block">
            <h3 class="preview-label">ตัวอย่าง Desktop</h3>
            <div class="preview-desktop">
              <div class="desktop-left" :style="{ backgroundColor: form.backgroundColor }">
                <img v-if="form.logoImage" :src="form.logoImage" class="d-logo" />
                <div v-else class="d-logo-placeholder">asap</div>
                <h2 class="d-title">{{ form.title || 'จองสะดวกขึ้นผ่านแอปเอแซ็ป' }}</h2>
                <p class="d-desc">{{ form.description || 'ค้นหารถเช่า check-in สะสม ASC Coin ผ่านแอป' }}</p>
                <div class="d-buttons">
                  <div v-if="form.showAppStore" class="store-btn">
                    <el-icon><Apple /></el-icon>
                    <div class="store-btn-text">
                      <span class="store-small">Download on the</span>
                      <span class="store-large">App Store</span>
                    </div>
                  </div>
                  <div v-if="form.showGooglePlay" class="store-btn">
                    <el-icon><Promotion /></el-icon>
                    <div class="store-btn-text">
                      <span class="store-small">GET IT ON</span>
                      <span class="store-large">Google Play</span>
                    </div>
                  </div>
                </div>
              </div>
              <div class="desktop-right">
                <img v-if="form.bannerImage" :src="form.bannerImage" class="d-banner-img" />
                <div v-else class="d-placeholder">
                  <el-icon><Picture /></el-icon>
                  <span>รูปภาพ Desktop</span>
                </div>
              </div>
            </div>
          </div>

          <!-- Mobile Preview -->
          <div class="preview-block">
            <h3 class="preview-label">ตัวอย่าง Mobile</h3>
            <div class="preview-mobile-wrapper">
              <div class="preview-mobile">
                <!-- Top: phone image on gradient bg -->
                <div class="m-top" :style="{ background: `linear-gradient(180deg, ${form.backgroundColor}55 0%, ${form.backgroundColor} 100%)` }">
                  <img v-if="form.bannerImageMobile" :src="form.bannerImageMobile" class="m-phone-img" />
                  <div v-else class="m-placeholder">
                    <el-icon><Picture /></el-icon>
                    <span>รูปภาพ Mobile</span>
                  </div>
                </div>
                <!-- Bottom: logo + text + store buttons -->
                <div class="m-bottom" :style="{ backgroundColor: form.backgroundColor }">
                  <img v-if="form.logoImage" :src="form.logoImage" class="m-logo" />
                  <div v-else class="m-logo-placeholder">asap</div>
                  <h2 class="m-title">{{ form.title || 'Easy booking' }}</h2>
                  <p class="m-desc">{{ form.description || "Let's go" }}</p>
                  <div class="m-buttons">
                    <div v-if="form.showAppStore" class="store-btn store-btn--dark">
                      <el-icon><Apple /></el-icon>
                      <div class="store-btn-text">
                        <span class="store-small">Download on the</span>
                        <span class="store-large">App Store</span>
                      </div>
                    </div>
                    <div v-if="form.showGooglePlay" class="store-btn store-btn--dark">
                      <el-icon><Promotion /></el-icon>
                      <div class="store-btn-text">
                        <span class="store-small">GET IT ON</span>
                        <span class="store-large">Google Play</span>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

        </div>

        <el-divider />

        <!-- ===== IMAGES ===== -->
        <el-row :gutter="24">
          <el-col :span="8">
            <el-form-item label="รูปภาพแบนเนอร์ Desktop" prop="bannerImage">
              <div class="image-upload-wrapper">
                <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false"
                  :on-change="(f) => handleImageChange(f, 'bannerImage')" accept="image/*">
                  <div v-if="form.bannerImage" class="uploaded-image">
                    <el-image :src="form.bannerImage" fit="contain" />
                    <div class="image-actions">
                      <el-button type="danger" size="small" @click.stop="form.bannerImage = ''">
                        <el-icon><Delete /></el-icon> ลบรูป
                      </el-button>
                    </div>
                  </div>
                  <div v-else class="upload-placeholder">
                    <el-icon class="upload-icon"><Plus /></el-icon>
                    <span>อัปโหลดรูป Desktop</span>
                    <p class="upload-hint">แนะนำขนาด 600 x 400 px</p>
                  </div>
                </el-upload>
              </div>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item label="รูปภาพแบนเนอร์ Mobile" prop="bannerImageMobile">
              <div class="image-upload-wrapper">
                <el-upload class="image-uploader" :show-file-list="false" :auto-upload="false"
                  :on-change="(f) => handleImageChange(f, 'bannerImageMobile')" accept="image/*">
                  <div v-if="form.bannerImageMobile" class="uploaded-image">
                    <el-image :src="form.bannerImageMobile" fit="contain" />
                    <div class="image-actions">
                      <el-button type="danger" size="small" @click.stop="form.bannerImageMobile = ''">
                        <el-icon><Delete /></el-icon> ลบรูป
                      </el-button>
                    </div>
                  </div>
                  <div v-else class="upload-placeholder">
                    <el-icon class="upload-icon"><Plus /></el-icon>
                    <span>อัปโหลดรูป Mobile</span>
                    <p class="upload-hint">แนะนำขนาด 390 x 360 px</p>
                  </div>
                </el-upload>
              </div>
            </el-form-item>
          </el-col>

          <el-col :span="8">
            <el-form-item label="โลโก้แอป" prop="logoImage">
              <div class="image-upload-wrapper">
                <el-upload class="image-uploader logo-uploader" :show-file-list="false" :auto-upload="false"
                  :on-change="(f) => handleImageChange(f, 'logoImage')" accept="image/*">
                  <div v-if="form.logoImage" class="uploaded-image small">
                    <el-image :src="form.logoImage" fit="contain" />
                    <div class="image-actions">
                      <el-button type="danger" size="small" @click.stop="form.logoImage = ''">
                        <el-icon><Delete /></el-icon>
                      </el-button>
                    </div>
                  </div>
                  <div v-else class="upload-placeholder small">
                    <el-icon class="upload-icon"><Plus /></el-icon>
                    <span>โลโก้แอป</span>
                    <p class="upload-hint">100 x 100 px</p>
                  </div>
                </el-upload>
              </div>
            </el-form-item>
          </el-col>
        </el-row>

        <!-- ===== CONTENT ===== -->
        <el-row :gutter="24">
          <el-col :span="12">
            <el-form-item label="หัวข้อ" prop="title">
              <el-input v-model="form.title" placeholder="เช่น Easy booking" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="สีพื้นหลัง (Desktop)" prop="backgroundColor">
              <div style="display:flex; align-items:center; gap:12px;">
                <el-color-picker v-model="form.backgroundColor" show-alpha />
                <el-input v-model="form.backgroundColor" style="width: 150px;" />
              </div>
            </el-form-item>
          </el-col>
        </el-row>

        <el-form-item label="คำอธิบาย" prop="description">
          <el-input v-model="form.description" type="textarea" :rows="2"
            placeholder="เช่น ค้นหารถเช่า check-in สะสม ASC Coin ผ่านแอป" />
        </el-form-item>

        <el-row :gutter="24">
          <el-col :span="12">
            <el-form-item label="ลิงก์ App Store (iOS)" prop="appStoreLink">
              <div style="display:flex; align-items:center; gap:12px;">
                <el-input v-model="form.appStoreLink" placeholder="https://apps.apple.com/..."
                  :disabled="!form.showAppStore">
                  <template #prefix><el-icon><Link /></el-icon></template>
                </el-input>
                <el-switch v-model="form.showAppStore" active-text="เปิด" inactive-text="ปิด" inline-prompt />
              </div>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="ลิงก์ Google Play (Android)" prop="googlePlayLink">
              <div style="display:flex; align-items:center; gap:12px;">
                <el-input v-model="form.googlePlayLink" placeholder="https://play.google.com/..."
                  :disabled="!form.showGooglePlay">
                  <template #prefix><el-icon><Link /></el-icon></template>
                </el-input>
                <el-switch v-model="form.showGooglePlay" active-text="เปิด" inactive-text="ปิด" inline-prompt />
              </div>
            </el-form-item>
          </el-col>
        </el-row>

        <el-form-item label="สถานะ">
          <el-switch v-model="form.isActive" inline-prompt active-text="แสดง" inactive-text="ซ่อน" />
        </el-form-item>

        <el-divider />

        <div class="form-actions">
          <el-button type="primary" size="large" :loading="saving" :disabled="!isDirty || saving" @click="handleSubmit">
            <el-icon><Check /></el-icon>
            <span>บันทึกการเปลี่ยนแปลง</span>
          </el-button>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, watch, nextTick } from 'vue'
import { ElMessage } from 'element-plus'
import { useApi } from '@/composables/useApi'

const { getAppBanner, updateAppBanner } = useApi()

const loading = ref(false)
const saving = ref(false)
const isDirty = ref(false)
const formRef = ref()

const form = reactive({
  title: '',
  description: '',
  bannerImage: '',
  bannerImageMobile: '',
  logoImage: '',
  backgroundColor: '#FF595A',
  appStoreLink: '',
  googlePlayLink: '',
  showAppStore: true,
  showGooglePlay: true,
  isActive: true
})

watch(() => form, () => { isDirty.value = true }, { deep: true })

const rules = {
  title: [{ required: true, message: 'กรุณากรอกหัวข้อ', trigger: 'blur' }]
}

const handleImageChange = (file, field) => {
  const reader = new FileReader()
  reader.onload = (e) => { form[field] = e.target.result }
  reader.readAsDataURL(file.raw)
}

const fetchData = async () => {
  loading.value = true
  try {
    const data = await getAppBanner()
    Object.assign(form, data)
    nextTick(() => { isDirty.value = false })
  } catch {
    ElMessage.error('ไม่สามารถโหลดข้อมูลได้')
  } finally {
    loading.value = false
  }
}

const handleSubmit = async () => {
  if (!formRef.value) return
  await formRef.value.validate(async (valid) => {
    if (valid) {
      saving.value = true
      try {
        await updateAppBanner(form)
        ElMessage.success('บันทึกสำเร็จ')
        isDirty.value = false
      } catch {
        ElMessage.error('เกิดข้อผิดพลาด')
      } finally {
        saving.value = false
      }
    }
  })
}

onMounted(() => fetchData())
</script>

<style lang="scss" scoped>
/* ===== PREVIEWS WRAPPER ===== */
.previews-wrapper {
  display: flex;
  gap: 32px;
  align-items: flex-start;
  flex-wrap: wrap;
  margin-bottom: 8px;
}

.preview-block {
  flex: 1;
  min-width: 280px;
}

.preview-label {
  font-size: 13px;
  color: #666;
  font-weight: 600;
  margin-bottom: 10px;
}

/* ===== DESKTOP PREVIEW ===== */
.preview-desktop {
  display: flex;
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid #eee;
  height: 200px;
}

.desktop-left {
  width: 340px;
  min-width: 200px;
  padding: 20px 24px;
  color: #fff;
  display: flex;
  flex-direction: column;
  justify-content: center;

  .d-logo {
    width: 36px;
    height: 36px;
    border-radius: 8px;
    object-fit: cover;
    margin-bottom: 8px;
  }

  .d-logo-placeholder {
    font-size: 12px;
    font-weight: 800;
    text-transform: uppercase;
    opacity: 0.8;
    margin-bottom: 8px;
  }

  .d-title {
    font-size: 15px;
    font-weight: 700;
    margin: 0 0 4px;
    line-height: 1.3;
  }

  .d-desc {
    font-size: 11px;
    opacity: 0.85;
    margin: 0 0 12px;
    line-height: 1.4;
  }

  .d-buttons {
    display: flex;
    gap: 6px;
    flex-wrap: wrap;
  }
}

.desktop-right {
  flex: 1;
  background: #f0f0f0;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;

  .d-banner-img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .d-placeholder {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 6px;
    color: #ccc;
    font-size: 12px;

    .el-icon { font-size: 32px; }
  }
}

/* ===== MOBILE PREVIEW ===== */
.preview-mobile-wrapper {
  display: flex;
  justify-content: center;
}

.preview-mobile {
  width: 260px;
  border-radius: 20px;
  overflow: hidden;
  border: 1px solid #ddd;
  box-shadow: 0 4px 16px rgba(0,0,0,0.12);

  .m-top {
    height: 220px;
    display: flex;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    position: relative;

    .m-phone-img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }

    .m-placeholder {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 6px;
      color: rgba(255,255,255,0.6);
      font-size: 12px;

      .el-icon { font-size: 32px; }
    }
  }

  .m-bottom {
    padding: 20px 16px;
    display: flex;
    flex-direction: column;
    align-items: center;
    text-align: center;

    .m-logo {
      width: 52px;
      height: 52px;
      border-radius: 12px;
      object-fit: cover;
      margin-bottom: 10px;
    }

    .m-logo-placeholder {
      width: 52px;
      height: 52px;
      border-radius: 12px;
      background: #FF595A;
      color: #fff;
      font-size: 10px;
      font-weight: 800;
      text-transform: uppercase;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: 10px;
    }

    .m-title {
      font-size: 16px;
      font-weight: 700;
      color: #fff;
      margin: 0 0 4px;
    }

    .m-desc {
      font-size: 11px;
      color: rgba(255,255,255,0.6);
      margin: 0 0 14px;
    }

    .m-buttons {
      display: flex;
      flex-direction: column;
      gap: 8px;
      width: 100%;
    }
  }
}

/* ===== STORE BUTTONS ===== */
.store-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: #111;
  color: #fff;
  border-radius: 6px;
  padding: 5px 10px;
  font-size: 10px;

  .el-icon { font-size: 16px; flex-shrink: 0; }

  &--dark {
    width: 100%;
    justify-content: center;
    padding: 7px 12px;
  }
}

.store-btn-text {
  display: flex;
  flex-direction: column;
  line-height: 1.2;

  .store-small { font-size: 8px; opacity: 0.8; }
  .store-large { font-size: 11px; font-weight: 600; }
}

/* ===== IMAGE UPLOAD ===== */
.image-upload-wrapper {
  width: 100%;
}

.image-uploader {
  width: 100%;
  :deep(.el-upload) { width: 100%; }

  &.logo-uploader {
    :deep(.el-upload) { width: 150px; }
  }
}

.uploaded-image {
  width: 100%;
  height: 180px;
  position: relative;
  border: 1px solid #dcdfe6;
  border-radius: 8px;
  overflow: hidden;
  background: #fafafa;

  &.small {
    width: 150px;
    height: 150px;
  }

  .el-image { width: 100%; height: 100%; }

  .image-actions {
    position: absolute;
    bottom: 8px;
    right: 8px;
  }
}

.upload-placeholder {
  width: 100%;
  height: 180px;
  border: 2px dashed #dcdfe6;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #999;
  cursor: pointer;
  transition: border-color 0.3s;

  &.small {
    width: 150px;
    height: 150px;
  }

  &:hover { border-color: #FF595A; }

  .upload-icon { font-size: 32px; margin-bottom: 8px; }
  .upload-hint { font-size: 12px; margin-top: 8px; color: #bbb; }
}

/* ===== MISC ===== */
.form-actions {
  display: flex;
  justify-content: flex-end;
}

:deep(.el-button) {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  .el-icon { margin: 0; }
}
</style>
