<template>
  <a-row :wrap="false">
    <a-col flex="200px">
      <RouterLink to="/">
        <div class="title-bar">
          <img class="logo" src="../assets/logo.png" alt="logo" />
          <div class="title">云深壁纸</div>
        </div>
      </RouterLink>
    </a-col>
    <a-col flex="auto">
      <a-menu
        v-model:selectedKeys="current"
        mode="horizontal"
        :items="items"
        @click="doMenuClick"
      />
    </a-col>
    <!-- 用户信息展示 -->
    <a-col flex="120px">
      <div v-if="loginUserStore.loginUser.id">
        <a-dropdown>
          <ASpace>
            <a-avatar :src="loginUserStore.loginUser.userAvatar" />
            {{ loginUserStore.loginUser.userName ?? '无名' }}
          </ASpace>
          <template #overlay>
            <a-menu>
              <a-menu-item @click="router.push('/user/profile')">
                <UserOutlined />
                个人中心
              </a-menu-item>
              <a-menu-item>
                <router-link to="/my_space">
                  <FolderOutlined />
                  我的空间
                </router-link>
              </a-menu-item>
              <a-menu-item @click="doLogout">
                <LogoutOutlined />
                退出登录
              </a-menu-item>
            </a-menu>
          </template>
        </a-dropdown>
      </div>
      <div v-else>
        <a-space>
          <a-button type="primary" @click="router.push('/user/login')">登录</a-button>
          <a-button @click="router.push('/user/register')">注册</a-button>
        </a-space>
      </div>
    </a-col>
  </a-row>
  <!-- 隐藏的图片组件用于预览联系方式 -->
  <a-image
    :src="contactImageUrl"
    :preview="{ visible: previewVisible, onVisibleChange: (visible) => previewVisible = visible }"
    style="display: none"
  />
</template>

<script lang="ts" setup>
import { computed, h, ref } from 'vue'
import { HomeOutlined, LogoutOutlined, UserOutlined, FolderOutlined } from '@ant-design/icons-vue'
import { message } from 'ant-design-vue'
import type { MenuProps } from 'ant-design-vue'

const loginUserStore = useLoginUserStore()

// 菜单列表
const originItems = [
  {
    key: '/',
    icon: () => h(HomeOutlined),
    label: '主页',
    title: '主页',
  },
  {
    key: '/add_picture',
    label: '创建图片',
    title: '创建图片',
  },
  {
    key: '/admin/userManage',
    label: '用户管理',
    title: '用户管理',
  },
  {
    key: '/admin/pictureManage',
    label: '图片管理',
    title: '图片管理',
  },
  {
    key: '/admin/spaceManage',
    label: '空间管理',
    title: '空间管理',
  },
  {
    key: 'others',
    label: '联系云深',
    title: '联系云深',
  },
]

// 过滤菜单项
const filterMenus = (menus = [] as MenuProps['items']) => {
  return menus?.filter((menu) => {
    if (menu && typeof menu.key === 'string' && menu.key.startsWith('/admin')) {
      const loginUser = loginUserStore.loginUser
      if (!loginUser || loginUser.userRole !== 'admin') {
        return false
      }
    }
    return true
  })
}

// 展示在菜单的路由数组
const items = computed<MenuProps['items']>(() => filterMenus(originItems))

import { useRouter } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { userLogoutUsingPost } from '@/api/userController.ts'
const router = useRouter()

// 联系方式图片
const contactImageUrl = 'https://yunshen-1359852853.cos.ap-shanghai.myqcloud.com/public/1969313712637472770/2025-10-19_9epMeagexFGK4ZSD.webp'
const previewVisible = ref(false)

// 路由跳转事件
const doMenuClick = ({ key }: { key: string }) => {
  // 如果点击的是"联系云深"菜单，显示联系方式图片
  if (key === 'others') {
    previewVisible.value = true
    return
  }
  router.push({
    path: key,
  })
}

// 当前选中菜单
const current = ref<string[]>([])
// 监听路由变化，更新当前选中菜单 eg:点击主页、关于等，刷新后保持高亮
router.afterEach((to) => {
  current.value = [to.path]
})

// 用户注销
const doLogout = async () => {
  const res = await userLogoutUsingPost()
  console.log(res)
  if (res.data.code === 0) {
    loginUserStore.setLoginUser({
      userName: '未登录',
    })
    message.success('退出登录成功')
    await router.push('/user/login')
  } else {
    message.error('退出登录失败，' + res.data.message)
  }
}
</script>

<style scoped>
.title-bar {
  display: flex;
  align-items: center;
}

.title {
  color: black;
  font-size: 18px;
  margin-left: 16px;
}

.logo {
  height: 48px;
}
</style>
