<template>
  <div id="searchPicturePage">
    <h2 style="margin-bottom: 16px">以图搜图</h2>
    <h3 style="margin: 16px 0">原图</h3>
    <a-card style="width: 240px">
      <template #cover>
        <img
          style="height: 180px; object-fit: cover"
          :alt="picture.name"
          :src="picture.thumbnailUrl ?? picture.url"
        />
      </template>
    </a-card>
    <h3 style="margin: 16px 0">识图结果</h3>
    <!-- 图片列表 -->
    <a-list
      :grid="{ gutter: 16, xs: 1, sm: 2, md: 3, lg: 4, xl: 5, xxl: 6 }"
      :data-source="dataList"
    >
      <template #renderItem="{ item }">
        <a-list-item style="padding: 0">
          <a :href="item.fromUrl" target="_blank">
            <a-card>
              <template #cover>
                <img style="height: 180px; object-fit: cover" :src="item.thumbUrl" />
              </template>
            </a-card>
          </a>
        </a-list-item>
      </template>
    </a-list>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import { useRoute } from 'vue-router'
import { getPictureVoByIdUsingGet, searchPictureByPictureUsingPost } from '@/api/pictureController'
import { message } from 'ant-design-vue'

const route = useRoute()

// 图片 id
const pictureId = computed(() => {
  return route.query?.pictureId
})

const picture = ref<API.PictureVO>({})
const dataList = ref<API.ImageSearchResult[]>([])

// 获取原图数据
const getOldPicture = async () => {
  console.log('[以图搜图] 页面接收到的 pictureId:', pictureId.value)

  if (!pictureId.value) {
    console.warn('[以图搜图] 警告: pictureId 为空')
    return
  }

  // 保持字符串格式，不转换为数字，避免大整数精度丢失
  const id = pictureId.value
  console.log('[以图搜图] 使用的 ID (字符串):', id)

  try {
    const res = await getPictureVoByIdUsingGet({
      id: id as any,
    })

    if (res.data.code === 0 && res.data.data) {
      picture.value = res.data.data
    } else {
      console.warn('[以图搜图] 原图数据获取失败，但不影响搜索:', res.data.message)
    }
  } catch (error: any) {
    console.error('[以图搜图] 获取原图数据异常:', error)
    // 不显示错误提示，静默失败
  }
}

// 获取搜图结果
const fetchSearchResult = async () => {
  if (!pictureId.value) {
    message.error('图片ID不存在')
    console.error('[以图搜图] 错误: 搜索时 pictureId 为空')
    return
  }

  // 保持字符串格式，不转换为数字，避免大整数精度丢失
  const id = pictureId.value
  console.log('[以图搜图] 准备搜索，ID (字符串):', id)

  try {
    console.log('[以图搜图] 开始搜索相似图片, ID:', id)
    const res = await searchPictureByPictureUsingPost({
      pictureId: id as any,
    })
    console.log('[以图搜图] 搜索结果响应:', res.data)

    if (res.data.code === 0 && res.data.data) {
      dataList.value = res.data.data ?? []
      console.log('[以图搜图] 搜索成功，找到', dataList.value.length, '个结果')
    } else {
      message.error('获取数据失败，' + res.data.message)
      console.error('[以图搜图] 搜索失败:', res.data.message)
    }
  } catch (error: any) {
    message.error('搜索失败: ' + error.message)
    console.error('[以图搜图] 搜索异常:', error)
  }
}

// 页面加载时请求数据
onMounted(() => {
  console.log('[以图搜图] 页面加载，route.query:', route.query)
  getOldPicture()
  fetchSearchResult()
})
</script>
