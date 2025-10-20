<template>
  <div id="basicLayout">
    <a-layout style="min-height: 100vh">
      <a-layout-header class="header">
        <GlobalHeader />
      </a-layout-header>
      <a-layout>
        <GlobalSider class="sider" />
        <a-layout-content class="content">
          <router-view />
        </a-layout-content>
      </a-layout>
      <a-layout-footer class="footer">
        <a @click.prevent="showContactImage" style="cursor: pointer">
          云深壁纸 by 云深
        </a>
      </a-layout-footer>

    </a-layout>
    <!-- 隐藏的图片组件用于预览联系方式 -->
    <a-image
      :src="contactImageUrl"
      :preview="{ visible: previewVisible, onVisibleChange: (visible: boolean) => previewVisible = visible }"
      style="display: none"
    />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import GlobalHeader from '@/components/GlobalHeader.vue'
import GlobalSider from '@/components/GlobalSider.vue'

// 联系方式图片
const contactImageUrl = 'https://yunshen-1359852853.cos.ap-shanghai.myqcloud.com/public/1969313712637472770/2025-10-19_9epMeagexFGK4ZSD.webp'
const previewVisible = ref(false)

// 显示联系方式图片
const showContactImage = () => {
  previewVisible.value = true
}
</script>

<style scoped>
#basicLayout .header {
  padding-inline: 20px;
  margin-bottom: 1px;
  color: unset;
  background: white;
}

#basicLayout .content {
  background: linear-gradient(to right, #fefefe, #fff);
  margin-bottom: 28px;
  padding: 28px;
}

#basicLayout .sider {
  background: #fff;
  padding-top: 20px;
  border-right: 0.5px solid #eee;
}

#basicLayout :deep(.ant-menu-root) {
  border-bottom: none !important;
  border-inline-end: none !important;
}
#basicLayout .footer {
  background: #efefef;
  padding: 16px;
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  text-align: center;
}
</style>

