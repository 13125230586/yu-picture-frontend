<template>
  <div id="pictureDetailPage">
    <a-row :gutter="[16, 16]">
      <!-- 图片预览 -->
      <a-col :sm="24" :md="16" :xl="18">
        <a-card title="图片预览">
          <!-- 图片展示区域 -->
          <div style="display: flex; justify-content: center; align-items: center; min-height: 350px; margin-bottom: 16px;">
            <a-image :src="picture.url" style="max-height: 500px; object-fit: contain" />
          </div>
          
          <!-- 互动组件 -->
          <div class="picture-actions">
            <a-space size="large">
              <div class="action-item" @click="handleLike">
                <a-button type="text" size="large" :class="{ 'liked': isLiked }">
                  <HeartOutlined />
                </a-button>
                <div class="action-info">
                  <span class="action-count">{{ likeCount }}</span>
                  <span class="action-label">点赞</span>
                </div>
              </div>
              
              <div class="action-item" @click="doDownload">
                <a-button type="text" size="large">
                  <DownloadOutlined />
                </a-button>
                <div class="action-info">
                  <span class="action-count">{{ downloadCount }}</span>
                  <span class="action-label">下载</span>
                </div>
              </div>
              
              <div class="action-item" @click="handleCollect">
                <a-button type="text" size="large" :class="{ 'collected': isCollected }">
                  <StarOutlined />
                </a-button>
                <div class="action-info">
                  <span class="action-count">{{ collectCount }}</span>
                  <span class="action-label">收藏</span>
                </div>
              </div>
              
              <div class="action-item" @click="handleShare">
                <a-button type="text" size="large">
                  <ShareAltOutlined />
                </a-button>
                <div class="action-info">
                  <span class="action-count">{{ shareCount }}</span>
                  <span class="action-label">分享</span>
                </div>
              </div>
            </a-space>
          </div>
        </a-card>
      </a-col>
      <!-- 图片信息区域 -->
      <a-col :sm="24" :md="8" :xl="6">
        <a-card title="图片信息">
          <a-descriptions :column="1">
            <a-descriptions-item label="作者">
              <a-space>
                <a-avatar :size="24" :src="picture.user?.userAvatar" />
                <div>{{ picture.user?.userName }}</div>
              </a-space>
            </a-descriptions-item>
            <a-descriptions-item label="名称">
              {{ picture.name ?? '未命名' }}
            </a-descriptions-item>
            <a-descriptions-item label="简介">
              {{ picture.introduction ?? '-' }}
            </a-descriptions-item>
            <a-descriptions-item label="分类">
              {{ picture.category ?? '默认' }}
            </a-descriptions-item>
            <a-descriptions-item label="标签">
              <a-tag v-for="tag in picture.tags" :key="tag">
                {{ tag }}
              </a-tag>
            </a-descriptions-item>
            <a-descriptions-item label="格式">
              {{ picture.picFormat ?? '-' }}
            </a-descriptions-item>
            <a-descriptions-item label="宽度">
              {{ picture.picWidth ?? '-' }}
            </a-descriptions-item>
            <a-descriptions-item label="高度">
              {{ picture.picHeight ?? '-' }}
            </a-descriptions-item>
            <a-descriptions-item label="宽高比">
              {{ picture.picScale ?? '-' }}
            </a-descriptions-item>
            <a-descriptions-item label="大小">
              {{ formatSize(picture.picSize) }}
            </a-descriptions-item>
          </a-descriptions>
          <!-- 图片操作 -->
          <a-space wrap>
            <a-button type="primary" @click="doDownload">
              免费下载
              <template #icon>
                <DownloadOutlined />
              </template>
            </a-button>
            <a-button v-if="canEdit" :icon="h(EditOutlined)" type="default" @click="doEdit">
              编辑
            </a-button>
            <a-button v-if="canEdit" :icon="h(DeleteOutlined)" danger @click="doDelete">
              删除
            </a-button>
          </a-space>
        </a-card>
      </a-col>
    </a-row>
  </div>
</template>

<script setup lang="ts">
import { computed, h, onMounted, ref } from 'vue'
import { deletePictureUsingPost, getPictureVoByIdUsingGet } from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import { DeleteOutlined, DownloadOutlined, EditOutlined, HeartOutlined, StarOutlined, ShareAltOutlined } from '@ant-design/icons-vue'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { useRouter } from 'vue-router'
import { downloadImage, formatSize } from '@/utils'

interface Props {
  id: string | number
}

const props = defineProps<Props>()
const picture = ref<API.PictureVO>({})

// 互动状态
const isLiked = ref(false)
const isCollected = ref(false)

// 数量统计
const likeCount = ref(0)
const downloadCount = ref(0) 
const collectCount = ref(0)
const shareCount = ref(0)

const loginUserStore = useLoginUserStore()

// 是否具有编辑权限
const canEdit = computed(() => {
  const loginUser = loginUserStore.loginUser
  // 未登录不可编辑
  if (!loginUser.id) {
    return false
  }
  // 仅本人或管理员可编辑
  const user = picture.value.user || {}
  return loginUser.id === user.id || loginUser.userRole === 'admin'
})

// 获取图片详情
const fetchPictureDetail = async () => {
  try {
    const res = await getPictureVoByIdUsingGet({
      id: props.id,
    })
    if (res.data.code === 0 && res.data.data) {
      picture.value = res.data.data
    } else {
      message.error('获取图片详情失败，' + res.data.message)
    }
  } catch (e: any) {
    message.error('获取图片详情失败：' + e.message)
  }
}

onMounted(() => {
  fetchPictureDetail()
})

const router = useRouter()

// 编辑
const doEdit = () => {
  router.push('/add_picture?id=' + picture.value.id)
}

// 删除数据
const doDelete = async () => {
  const id = picture.value.id
  if (!id) {
    return
  }
  const res = await deletePictureUsingPost({ id })
  if (res.data.code === 0) {
    message.success('删除成功')
  } else {
    message.error('删除失败')
  }
}

// 下载图片
const doDownload = () => {
  downloadImage(picture.value.url)
  downloadCount.value++
  message.success('开始下载图片')
}

// 点赞
const handleLike = () => {
  isLiked.value = !isLiked.value
  if (isLiked.value) {
    likeCount.value++
    message.success('点赞成功')
  } else {
    likeCount.value--
    message.success('取消点赞')
  }
}

// 收藏
const handleCollect = () => {
  isCollected.value = !isCollected.value
  if (isCollected.value) {
    collectCount.value++
    message.success('收藏成功')
  } else {
    collectCount.value--
    message.success('取消收藏')
  }
}

// 分享
const handleShare = () => {
  if (navigator.share) {
    navigator.share({
      title: picture.value.name,
      text: picture.value.introduction,
      url: window.location.href
    })
  } else {
    navigator.clipboard.writeText(window.location.href).then(() => {
      message.success('链接已复制到剪贴板')
    }).catch(() => {
      message.error('分享失败')
    })
  }
  shareCount.value++
}
</script>

<style scoped>
#pictureDetailPage {
  margin-bottom: 16px;
}

.picture-actions {
  display: flex;
  justify-content: center;
  padding: 16px 0;
  border-top: 1px solid #f0f0f0;
}

.action-item {
  display: flex;
  align-items: center;
  cursor: pointer;
  padding: 6px 12px;
  border-radius: 8px;
  transition: all 0.3s ease;
  gap: 8px;
}

.action-info {
  display: flex;
  align-items: center;
  gap: 4px;
}

.action-item:hover {
  background-color: #f5f5f5;
}

.action-item :deep(.ant-btn) {
  border: none;
  box-shadow: none;
  font-size: 20px;
  height: auto;
  padding: 8px;
  color: #666;
}

.action-item :deep(.ant-btn:hover) {
  color: #1677ff;
}

.action-item :deep(.ant-btn.liked) {
  color: #ff4d4f;
}

.action-item :deep(.ant-btn.collected) {
  color: #faad14;
}

.action-count {
  font-size: 14px;
  font-weight: 500;
  color: #262626;
  line-height: 1;
}

.action-label {
  font-size: 12px;
  color: #8c8c8c;
  line-height: 1;
}

@media (max-width: 768px) {
  .picture-actions {
    padding: 6px 0;
  }
  
  .action-item {
    padding: 4px 8px;
    gap: 6px;
  }
  
  .action-item :deep(.ant-btn) {
    font-size: 18px;
  }
}
</style>

