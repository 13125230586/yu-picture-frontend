<template>
  <div id="globalSider">
    <!-- 自定义折叠按钮 -->
    <div v-if="loginUserStore.loginUser.id" class="toggle-btn" @click="collapsed = !collapsed">
      <MenuFoldOutlined v-if="!collapsed" />
      <MenuUnfoldOutlined v-if="collapsed" />
    </div>

    <a-layout-sider v-if="loginUserStore.loginUser.id"
                    v-model:collapsed="collapsed"
                    class="sider"
                    width="200"
                    collapsed-width="0"
                    :trigger="null"
                    :default-collapsed="true"
    >
      <a-menu
        v-model:selectedKeys="current"
        mode="inline"
        :items="menuItems"
        @click="doMenuClick"
      />
    </a-layout-sider>
  </div>
</template>
<script lang="ts" setup>
import { h, ref, watch } from 'vue'
import { PictureOutlined, UserOutlined, MenuFoldOutlined, MenuUnfoldOutlined } from '@ant-design/icons-vue'
import { useRouter } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'

const loginUserStore = useLoginUserStore()

// 侧边栏折叠状态，默认折叠
const collapsed = ref(true)

// 监听折叠状态变化，触发窗口 resize 事件以重新计算布局
watch(collapsed, () => {
  // 多次触发确保布局正确计算
  // 立即触发
  window.dispatchEvent(new Event('resize'))
  // 动画进行中触发
  setTimeout(() => window.dispatchEvent(new Event('resize')), 100)
  // 动画完成后触发
  setTimeout(() => window.dispatchEvent(new Event('resize')), 250)
  // 最后再确认一次
  setTimeout(() => window.dispatchEvent(new Event('resize')), 400)
})

// 菜单项
const menuItems = [
  {
    key: '/',
    icon: () => h(PictureOutlined),
    label: '公共图库',
  },
  {
    key: '/my_space',
    label: '我的空间',
    icon: () => h(UserOutlined),
  },
]

const router = useRouter()
// 当前要高亮的菜单项
const current = ref<string[]>([])
// 监听路由变化，更新高亮菜单项
router.afterEach((to, from, next) => {
  current.value = [to.path]
})

// 路由跳转事件
const doMenuClick = ({ key }) => {
  router.push({
    path: key,
  })
}
</script>

<style scoped>
#globalSider {
  position: relative;
}

#globalSider .ant-layout-sider {
  background: none;
}

.toggle-btn {
  position: fixed;
  left: 10px;
  top: 80px;
  z-index: 1000;
  width: 40px;
  height: 40px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 18px;
  color: #666;
}

.toggle-btn:hover {
  background: #1677ff;
  color: white;
  box-shadow: 0 4px 12px rgba(22, 119, 255, 0.3);
}
</style>

