<template>
  <div id="spaceDetailPage">
    <!-- 空间信息 -->
    <a-flex justify="space-between">
      <h2>{{ space.spaceName }}（私有空间）</h2>
      <a-space size="middle">
        <a-button type="primary" :href="`/add_picture?spaceId=${id}`" target="_blank">
          + 创建图片
        </a-button>
        <a-tooltip
          :title="`占用空间 ${formatSize(Math.max(0, space.totalSize || 0))} / ${formatSize(space.maxSize || 0)}`"
        >
          <a-progress
            type="circle"
            :size="42"
            :percent="spaceUsagePercent"
            :format="(percent) => `${percent.toFixed(1)}%`"
          />
        </a-tooltip>
      </a-space>
    </a-flex>
    <div style="margin-bottom: 16px" />

    <!-- 搜索框 -->
    <div class="search-bar">
      <a-input-search
        v-model:value="searchParams.name"
        placeholder="搜索图片名称"
        enter-button="搜索"
        size="large"
        @search="doSearch"
        allow-clear
      />
    </div>

    <!-- 图片列表 -->
    <SpacePictureList
      :pictureList="dataList"
      :loading="loading"
      :onReload="fetchData"
    />

    <!-- 分页 -->
    <div class="pagination-wrapper" v-if="total > 0">
      <a-pagination
        v-model:current="searchParams.current"
        v-model:page-size="searchParams.pageSize"
        :total="total"
        :show-size-changer="true"
        :page-size-options="['50', '100', '200']"
        @change="fetchData"
        @showSizeChange="fetchData"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, reactive, ref } from 'vue'
import { getSpaceVoByIdUsingGet } from '@/api/spaceController.ts'
import { message } from 'ant-design-vue'
import { listPictureVoByPageUsingPost } from '@/api/pictureController.ts'
import { formatSize } from '@/utils'
import SpacePictureList from '@/components/space/SpacePictureList.vue'

interface Props {
  id: string | number
}

const props = defineProps<Props>()
const space = ref<API.SpaceVO>({})

// 计算空间占用百分比
const spaceUsagePercent = computed(() => {
  // 处理负值和空值
  const totalSize = Math.max(0, space.value.totalSize || 0)
  const maxSize = space.value.maxSize || 1 // 避免除以0

  if (maxSize === 0 || totalSize === 0) return 0

  const percent = (totalSize / maxSize) * 100

  // 确保百分比在 0-100 之间
  if (percent < 0) return 0
  if (percent > 100) return 100

  // 返回数字，格式化交给 format 函数
  return percent
})

// -------- 获取空间详情 --------
const fetchSpaceDetail = async () => {
  try {
    const res = await getSpaceVoByIdUsingGet({
      id: props.id,
    })
    if (res.data.code === 0 && res.data.data) {
      space.value = res.data.data
    } else {
      message.error('获取空间详情失败，' + res.data.message)
    }
  } catch (e: any) {
    message.error('获取空间详情失败：' + e.message)
  }
}

onMounted(() => {
  fetchSpaceDetail()
})

// --------- 获取图片列表 --------

// 定义数据
const dataList = ref<API.PictureVO[]>([])
const total = ref(0)
const loading = ref(true)

// 搜索条件
const searchParams = reactive<API.PictureQueryRequest>({
  current: 1,
  pageSize: 50,
  sortField: 'createTime',
  sortOrder: 'descend',
  name: '', // 图片名称搜索
})

// 搜索
const doSearch = () => {
  // 重置到第一页
  searchParams.current = 1
  fetchData()
}

// 获取数据
const fetchData = async () => {
  loading.value = true
  // 转换搜索参数
  const params = {
    spaceId: props.id,
    ...searchParams,
  }
  const res = await listPictureVoByPageUsingPost(params)
  if (res.data.code === 0 && res.data.data) {
    dataList.value = res.data.data.records ?? []
    total.value = res.data.data.total ?? 0
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
  loading.value = false
}

// 页面加载时获取数据，请求一次
onMounted(() => {
  fetchData()
})
</script>

<style scoped>
#spaceDetailPage {
  margin-bottom: 16px;
}

.search-bar {
  max-width: 600px;
  margin: 0 auto 24px;
}

.pagination-wrapper {
  text-align: center;
  margin: 40px 0;
  padding: 20px;
}
</style>
