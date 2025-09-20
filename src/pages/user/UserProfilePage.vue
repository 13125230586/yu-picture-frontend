<template>
  <div id="userProfilePage">
    <a-card title="个人中心" style="max-width: 600px; margin: 0 auto">
      <a-form
        :model="formState"
        name="userProfile"
        :label-col="{ span: 6 }"
        :wrapper-col="{ span: 18 }"
        autocomplete="off"
        @finish="handleSubmit"
      >
        <a-form-item label="用户名" name="userName" :rules="[{ required: true, message: '请输入用户名' }]">
          <a-input v-model:value="formState.userName" placeholder="请输入用户名" />
        </a-form-item>
        
        <a-form-item label="头像" name="userAvatar">
          <div class="avatar-upload">
            <a-avatar :size="80" :src="formState.userAvatar" />
            <a-input
              v-model:value="formState.userAvatar"
              placeholder="请输入头像URL"
              style="margin-left: 16px"
            />
          </div>
        </a-form-item>
        
        <a-form-item label="个人简介" name="userProfile">
          <a-textarea
            v-model:value="formState.userProfile"
            placeholder="请输入个人简介"
            :rows="4"
            :maxlength="200"
            show-count
          />
        </a-form-item>
        
        <a-form-item label="账号">
          <a-input :value="loginUserStore.loginUser.userAccount" disabled />
        </a-form-item>
        
        <a-form-item label="用户角色">
          <a-tag v-if="loginUserStore.loginUser.userRole === 'admin'" color="green">管理员</a-tag>
          <a-tag v-else color="blue">普通用户</a-tag>
        </a-form-item>
        
        <a-form-item :wrapper-col="{ offset: 6, span: 18 }">
          <a-space>
            <a-button type="primary" html-type="submit" :loading="loading">保存修改</a-button>
            <a-button @click="resetForm">重置</a-button>
          </a-space>
        </a-form-item>
      </a-form>
    </a-card>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref, onMounted } from 'vue'
import { message } from 'ant-design-vue'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { updateUserUsingPost } from '@/api/userController.ts'

const loginUserStore = useLoginUserStore()
const loading = ref(false)

const formState = reactive<API.UserUpdateRequest>({
  id: '',
  userName: '',
  userAvatar: '',
  userProfile: '',
})

// 初始化表单数据
const initForm = () => {
  const loginUser = loginUserStore.loginUser
  formState.id = loginUser.id || ''
  formState.userName = loginUser.userName || ''
  formState.userAvatar = loginUser.userAvatar || ''
  formState.userProfile = loginUser.userProfile || ''
}

// 重置表单
const resetForm = () => {
  initForm()
}

// 提交表单
const handleSubmit = async (values: any) => {
  if (!formState.id) {
    message.error('用户信息获取失败，请重新登录')
    return
  }
  
  loading.value = true
  try {
    const res = await updateUserUsingPost(formState)
    if (res.data.code === 0) {
      message.success('个人信息更新成功')
      // 更新本地用户信息
      await loginUserStore.fetchLoginUser()
    } else {
      message.error('更新失败：' + res.data.message)
    }
  } catch (error) {
    message.error('更新失败，请重试')
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  initForm()
})
</script>

<style scoped>
#userProfilePage {
  padding: 20px;
}

.avatar-upload {
  display: flex;
  align-items: center;
}
</style>