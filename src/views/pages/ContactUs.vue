<template>
  <div class="contact-us">
    <div class="page-header">
      <h1>ข้อมูลติดต่อ</h1>
    </div>

    <el-form
      ref="formRef"
      :model="form"
      :rules="rules"
      label-position="top"
      class="content-card"
      v-loading="loading"
    >
      <div class="form-section">
        <div class="form-section-title">ข้อมูลติดต่อ</div>

        <el-form-item label="ชื่อบริษัท" prop="companyName">
          <el-input v-model="form.companyName" placeholder="กรอกชื่อบริษัท" />
        </el-form-item>

        <el-form-item label="ที่อยู่" prop="address">
          <el-input
            v-model="form.address"
            type="textarea"
            :rows="3"
            placeholder="กรอกที่อยู่"
          />
        </el-form-item>

        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="เบอร์โทรศัพท์" prop="phone">
              <el-input v-model="form.phone" placeholder="02-xxx-xxxx" />
            </el-form-item>
          </el-col>
        </el-row>
      </div>

      <div class="form-section">
        <div class="form-section-title">โซเชียลมีเดีย</div>

        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="LINE ชื่อ" prop="lineId">
              <el-input v-model="form.lineId" placeholder="@asapcarrental" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="LINE Link" prop="lineUrl">
              <el-input v-model="form.lineUrl" placeholder="https://line.me/..." />
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="Facebook ชื่อเพจ" prop="facebook">
              <el-input v-model="form.facebook" placeholder="asap Car Rental" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Facebook Link" prop="facebookUrl">
              <el-input v-model="form.facebookUrl" placeholder="https://facebook.com/..." />
            </el-form-item>
          </el-col>
        </el-row>
      </div>

      <div class="form-actions">
        <el-button type="primary" :loading="saving" :disabled="!isDirty || saving" @click="handleSubmit">
          บันทึกข้อมูล
        </el-button>
      </div>
    </el-form>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, watch, nextTick } from 'vue'
import { ElMessage } from 'element-plus'
import { useApi } from '@/composables/useApi'

const { getContactUs, updateContactUs } = useApi()

const formRef = ref()
const loading = ref(false)
const saving = ref(false)
const isDirty = ref(false)

const form = reactive({
  companyName: '',
  address: '',
  phone: '',
  lineId: '',
  lineUrl: '',
  facebook: '',
  facebookUrl: ''
})

watch(() => form, () => { isDirty.value = true }, { deep: true })

const rules = {
  address: [{ required: true, message: 'กรุณากรอกที่อยู่', trigger: 'blur' }]
}

const loadData = async () => {
  loading.value = true
  try {
    const data = await getContactUs()
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
        await updateContactUs(form)
        ElMessage.success('บันทึกข้อมูลสำเร็จ')
        isDirty.value = false
      } catch {
        ElMessage.error('เกิดข้อผิดพลาด กรุณาลองใหม่')
      } finally {
        saving.value = false
      }
    }
  })
}

onMounted(() => {
  loadData()
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

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  padding-top: 24px;
  border-top: 1px solid #eee;
}
</style>
