<template>
  <login-layout>
    <LoginContainer>
      <h4 class="mb-16">忘记密码</h4>
      <el-form
        class="register-form"
        ref="resetPasswordFormRef"
        :model="CheckEmailForm"
        :rules="rules"
      >
        <el-form-item prop="email">
          <el-input
            size="large"
            class="input-item"
            v-model="CheckEmailForm.email"
            placeholder="请输入邮箱"
          >
            <template #prepend>
              <el-button icon="UserFilled" />
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="code">
          <div class="flex-between w-full">
            <el-input
              size="large"
              class="code-input"
              v-model="CheckEmailForm.code"
              placeholder="请输入验证码"
            >
              <template #prepend>
                <el-button icon="Key" />
              </template>
            </el-input>
            <el-button
              size="large"
              class="send-email-button ml-16"
              @click="sendEmail"
              :loading="loading"
              >获取验证码</el-button
            >
          </div>
        </el-form-item>
      </el-form>
      <el-button type="primary" class="login-submit-button w-full" @click="checkCode"
        >立即验证</el-button
      >
      <div class="operate-container mt-8">
        <el-button
          class="register"
          @click="router.push('/login')"
          link
          type="primary"
          icon="DArrowLeft"
        >
          返回登录
        </el-button>
      </div>
    </LoginContainer>
  </login-layout>
</template>
<script setup lang="ts">
import { ref } from 'vue'
import type { CheckCodeRequest } from '@/api/type/user'
import { useRouter } from 'vue-router'
import type { FormInstance, FormRules } from 'element-plus'
import UserApi from '@/api/user'
import { MsgSuccess } from '@/utils/message'

const router = useRouter()
const CheckEmailForm = ref<CheckCodeRequest>({
  email: '',
  code: '',
  type: 'reset_password'
})

const resetPasswordFormRef = ref<FormInstance>()
const rules = ref<FormRules<CheckCodeRequest>>({
  email: [
    { required: true, message: '请输入邮箱', trigger: 'blur' },
    {
      validator: (rule, value, callback) => {
        const emailRegExp = /^[a-zA-Z0-9_.-]+@[a-zA-Z0-9-]+(\.[a-zA-Z0-9-]+)*\.[a-zA-Z0-9]{2,6}$/
        if (!emailRegExp.test(value) && value != '') {
          callback(new Error('请输入有效邮箱格式！'))
        } else {
          callback()
        }
      },
      trigger: 'blur'
    }
  ],
  code: [{ required: true, message: '请输入验证码' }]
})
const loading = ref<boolean>(false)

const checkCode = () => {
  resetPasswordFormRef.value
    ?.validate()
    .then(() => UserApi.checkCode(CheckEmailForm.value, loading))
    .then(() => router.push({ name: 'reset_password', params: CheckEmailForm.value }))
}
/**
 * 发送验证码
 */
const sendEmail = () => {
  resetPasswordFormRef.value?.validateField('email', (v: boolean) => {
    if (v) {
      UserApi.sendEmit(CheckEmailForm.value.email, 'reset_password', loading).then(() => {
        MsgSuccess('发送验证码成功')
      })
    }
  })
}
</script>
<style lang="scss" scope>
@import '../index.scss';
</style>