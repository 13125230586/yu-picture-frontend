<template>
  <div class="home-picture-list">
    <div
      ref="containerRef"
      class="masonry-container"
      :style="{
        height: `${containerHeight}px`,
      }"
    >
      <div
        v-for="picture in pictureList"
        :key="picture.id"
        class="picture-item"
        @click="handlePictureClick(picture)"
      >
        <!-- 图片容器 -->
        <div class="picture-wrapper">
          <img
            :src="picture.url"
            :alt="picture.name"
            class="picture-image"
            @load="handleImageLoad"
            @error="handleImageError"
          />

          <!-- 遮罩层 -->
          <div class="picture-overlay">
            <!-- 图片信息 -->
            <div class="picture-info-overlay">
              <div class="picture-title">{{ picture.name }}</div>
              <div class="picture-tags" v-if="picture.tags && picture.tags.length > 0">
                <a-tag
                  v-for="tag in picture.tags.slice(0, 3)"
                  :key="tag"
                  size="small"
                  class="tag-item"
                >
                  {{ tag }}
                </a-tag>
              </div>
              <div class="picture-meta">
                <span class="picture-size">{{ picture.picWidth }}×{{ picture.picHeight }}</span>
                <span class="picture-format">{{ picture.picFormat?.toUpperCase() }}</span>
              </div>
            </div>
            
            <!-- 操作按钮 -->
            <div class="picture-actions" @click.stop>
              <a-button
                v-if="showView"
                type="text"
                size="small"
                class="action-btn"
                @click="handleView(picture)"
                title="查看详情"
              >
                <EyeOutlined />
              </a-button>

              <a-button
                v-if="showDownload"
                type="text"
                size="small"
                class="action-btn"
                @click.stop.prevent="handleDownload(picture)"
                title="下载"
              >
                <DownloadOutlined />
              </a-button>

              <a-button
                v-if="showLike"
                type="text"
                size="small"
                class="action-btn"
                @click="handleLike(picture)"
                title="点赞"
              >
                <HeartOutlined />
              </a-button>

              <a-button
                v-if="showCollect"
                type="text"
                size="small"
                class="action-btn"
                @click="handleCollect(picture)"
                title="收藏"
              >
                <StarOutlined />
              </a-button>

              <a-button
                v-if="showShare"
                type="text"
                size="small"
                class="action-btn"
                @click="handleShare(picture)"
                title="分享"
              >
                <ShareAltOutlined />
              </a-button>

              <a-button
                v-if="showSearch"
                type="text"
                size="small"
                class="action-btn"
                @click="handleSearchSimilar(picture)"
                title="相似图片"
              >
                <SearchOutlined />
              </a-button>
            </div>
          </div>
        </div>

      </div>
    </div>

    <!-- 空状态 -->
    <div v-if="!loading && pictureList.length === 0" class="empty-state">
      <a-empty description="暂无图片" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick, watch } from 'vue'
import { message } from 'ant-design-vue'
import {
  EyeOutlined,
  DownloadOutlined,
  HeartOutlined,
  StarOutlined,
  ShareAltOutlined,
  SearchOutlined
} from '@ant-design/icons-vue'
import { downloadImage } from '@/utils/index'

interface Props {
  pictureList?: API.PictureVO[]
  loading?: boolean
  showView?: boolean
  showDownload?: boolean
  showLike?: boolean
  showCollect?: boolean
  showShare?: boolean
  showSearch?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  pictureList: () => [],
  loading: false,
  showView: true,
  showDownload: true,
  showLike: true,
  showCollect: true,
  showShare: true,
  showSearch: true
})

const containerRef = ref<HTMLElement>()
const pictureItems = ref<HTMLElement[]>([])

// 根据容器宽度计算列数
const columnCount = ref(4)
// 每列的高度数组
const columnHeights = ref<number[]>([])
// 容器总高度
const containerHeight = ref(0)
// 列宽度
const columnWidth = ref(0)
// 列间距
const gap = 16

const updateColumnCount = () => {
  if (!containerRef.value) return

  const containerWidth = containerRef.value.offsetWidth
  if (containerWidth < 576) {
    columnCount.value = 1 // 手机
  } else if (containerWidth < 768) {
    columnCount.value = 2 // 平板竖屏
  } else if (containerWidth < 1200) {
    columnCount.value = 3 // 平板横屏/小桌面
  } else if (containerWidth < 1600) {
    columnCount.value = 4 // 桌面
  } else {
    columnCount.value = 5 // 大屏
  }

  // 重新计算列宽和初始化列高度
  const totalGap = (columnCount.value - 1) * gap
  columnWidth.value = (containerWidth - totalGap) / columnCount.value
  columnHeights.value = new Array(columnCount.value).fill(0)
}

// 瀑布流布局计算
const calculateMasonryLayout = () => {
  if (!containerRef.value || !pictureItems.value.length) return

  // 重置列高度
  columnHeights.value = new Array(columnCount.value).fill(0)

  pictureItems.value.forEach((item) => {
    if (!item) return

    // 找到最短的列
    const shortestColumnIndex = columnHeights.value.indexOf(Math.min(...columnHeights.value))

    // 计算位置
    const left = shortestColumnIndex * (columnWidth.value + gap)
    const top = columnHeights.value[shortestColumnIndex]

    // 设置位置
    item.style.position = 'absolute'
    item.style.left = `${left}px`
    item.style.top = `${top}px`
    item.style.width = `${columnWidth.value}px`

    // 更新该列的高度
    const itemHeight = item.offsetHeight
    columnHeights.value[shortestColumnIndex] += itemHeight + gap
  })

  // 设置容器高度
  containerHeight.value = Math.max(...columnHeights.value)
}

// 图片加载完成
const handleImageLoad = () => {
  // 图片加载完成后重新计算布局
  nextTick(() => {
    calculateMasonryLayout()
  })
}

// 防抖函数
const debounce = (func: Function, wait: number) => {
  let timeout: number
  return function executedFunction(...args: any[]) {
    const later = () => {
      clearTimeout(timeout)
      func(...args)
    }
    clearTimeout(timeout)
    timeout = setTimeout(later, wait)
  }
}

// 防抖的布局重计算
const debouncedLayout = debounce(() => {
  updateColumnCount()
  nextTick(() => {
    calculateMasonryLayout()
  })
}, 100)

// 图片加载错误
const handleImageError = (event: Event) => {
  const img = event.target as HTMLImageElement
  img.src = '/src/assets/placeholder.png' // 替换为默认图片
}

// 点击图片
const handlePictureClick = (picture: API.PictureVO) => {
  handleView(picture)
}

// 查看图片
const handleView = (picture: API.PictureVO) => {
  window.open(`/picture/${picture.id}`, '_blank')
}

// 点赞
const handleLike = async (_picture: API.PictureVO) => {
  try {
    // TODO: 调用
    // 点赞API
    message.success('点赞成功')
  } catch (error) {
    message.error('点赞失败')
  }
}

// 收藏
const handleCollect = async (_picture: API.PictureVO) => {
  try {
    // TODO: 调用收藏API
    message.success('收藏成功')
  } catch (error) {
    message.error('收藏失败')
  }
}

// 分享
const handleShare = (picture: API.PictureVO) => {
  if (navigator.share) {
    navigator.share({
      title: picture.name,
      text: picture.introduction,
      url: window.location.origin + `/picture/${picture.id}`
    })
  } else {
    // 复制链接到剪贴板
    const url = window.location.origin + `/picture/${picture.id}`
    navigator.clipboard.writeText(url).then(() => {
      message.success('链接已复制到剪贴板')
    }).catch(() => {
      message.error('分享失败')
    })
  }
}

// 搜索相似图片
const handleSearchSimilar = (_picture: API.PictureVO) => {
  // TODO: 实现相似图片搜索
  message.info('相似图片搜索功能开发中')
}

// 下载图片
const handleDownload = (picture: API.PictureVO) => {
  if (picture.url) {
    downloadImage(picture.url, picture.name)
    message.success('开始下载图片')
  } else {
    message.error('图片链接不存在')
  }
}

// 监听图片列表变化
watch(() => props.pictureList, () => {
  nextTick(() => {
    // 重新获取所有图片项元素
    pictureItems.value = Array.from(containerRef.value?.querySelectorAll('.picture-item') || [])
    updateColumnCount()
    // 延迟执行布局计算，等待所有图片渲染完成
    setTimeout(() => {
      calculateMasonryLayout()
    }, 50)
  })
}, { flush: 'post' })

// 响应式监听
onMounted(() => {
  updateColumnCount()
  window.addEventListener('resize', debouncedLayout)

  // 初始化图片项
  nextTick(() => {
    pictureItems.value = Array.from(containerRef.value?.querySelectorAll('.picture-item') || [])
    // 延迟执行初始布局
    setTimeout(() => {
      calculateMasonryLayout()
    }, 100)
  })
})

onUnmounted(() => {
  window.removeEventListener('resize', debouncedLayout)
})
</script>

<style scoped>
.home-picture-list {
  width: 100%;
  padding: 0 16px;
}

.masonry-container {
  position: relative;
  width: 100%;
}

.picture-item {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  cursor: pointer;
}

.picture-item:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
}

.picture-wrapper {
  position: relative;
  overflow: hidden;
}

.picture-image {
  width: 100%;
  height: auto;
  display: block;
  transition: transform 0.3s ease;
}

.picture-item:hover .picture-image {
  transform: scale(1.05);
}

.picture-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(to bottom, transparent 0%, transparent 50%, rgba(0, 0, 0, 0.8) 100%);
  opacity: 0;
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  padding: 16px;
}

.picture-item:hover .picture-overlay {
  opacity: 1;
}

.picture-info-overlay {
  color: white;
  margin-bottom: 12px;
  transform: translateY(20px);
  transition: transform 0.3s ease;
}

.picture-item:hover .picture-info-overlay {
  transform: translateY(0);
}

.picture-actions {
  display: flex;
  gap: 8px;
  justify-content: center;
  transform: translateY(20px);
  transition: transform 0.3s ease 0.1s;
}

.picture-item:hover .picture-actions {
  transform: translateY(0);
}

.action-btn {
  background: rgba(255, 255, 255, 0.9) !important;
  border: none !important;
  border-radius: 50% !important;
  width: 40px !important;
  height: 40px !important;
  display: flex !important;
  align-items: center !important;
  justify-content: center !important;
  color: #666 !important;
  backdrop-filter: blur(4px);
}

.action-btn:hover {
  background: white !important;
  color: #1677ff !important;
  transform: scale(1.1);
}

.picture-info-overlay .picture-title {
  font-size: 14px;
  font-weight: 500;
  color: white;
  margin-bottom: 8px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.5);
}

.picture-info-overlay .picture-tags {
  margin-bottom: 8px;
}

.picture-info-overlay .tag-item {
  margin-right: 4px;
  margin-bottom: 4px;
  font-size: 12px;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: white;
}

.picture-info-overlay .picture-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.9);
}

.picture-info-overlay .picture-size, 
.picture-info-overlay .picture-format {
  background: rgba(255, 255, 255, 0.2);
  padding: 2px 6px;
  border-radius: 4px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: white;
}

.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: #999;
}

/* 响应式调整 */
@media (max-width: 576px) {
  .home-picture-list {
    padding: 0 8px;
  }

  .masonry-container {
    gap: 8px;
  }

  .picture-overlay {
    padding: 12px;
  }
  
  .picture-info-overlay .picture-title {
    font-size: 12px;
  }
  
  .picture-info-overlay .picture-meta {
    font-size: 10px;
  }

  .action-btn {
    width: 32px !important;
    height: 32px !important;
  }

  .picture-actions {
    gap: 6px;
  }
}

@media (max-width: 768px) {
  .picture-actions {
    gap: 6px;
  }

  .action-btn {
    width: 36px !important;
    height: 36px !important;
  }
  
  .picture-overlay {
    padding: 14px;
  }
}
</style>
