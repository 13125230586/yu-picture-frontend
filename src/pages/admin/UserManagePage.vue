<template>
  <div id="userManagePage">
    <!-- 搜索表单 -->
    <a-form layout="inline" :model="searchParams" @finish="doSearch">
      <a-form-item label="账号">
        <a-input v-model:value="searchParams.userAccount" placeholder="输入账号" allow-clear />
      </a-form-item>
      <a-form-item label="用户名">
        <a-input v-model:value="searchParams.userName" placeholder="输入用户名" allow-clear />
      </a-form-item>
      <a-form-item>
        <a-button type="primary" html-type="submit">搜索</a-button>
      </a-form-item>
    </a-form>
    <div style="margin-bottom: 16px" />
    <!-- 表格 -->
    <a-table
      :columns="columns"
      :data-source="dataList"
      :pagination="pagination"
      @change="doTableChange"
    >
      <template #headerCell="{ column }">
        <template v-if="column.dataIndex === 'createTime'">
          <div class="custom-header">
            创建时间
            <a-button type="text" size="small" @click="toggleSort" class="sort-button">
              <caret-up-outlined v-if="searchParams.sortOrder === 'ascend'" />
              <caret-down-outlined v-else />
            </a-button>
          </div>
        </template>
      </template>
      <template #bodyCell="{ column, record }">
        <template v-if="column.dataIndex === 'userAvatar'">
          <a-image :src="record.userAvatar" :width="120" />
        </template>
        <template v-else-if="column.dataIndex === 'userRole'">
          <div class="editable-cell">
            <div v-if="editableData[record.id]" class="editable-cell-input-wrapper">
              <a-select
                v-model:value="editableData[record.id].userRole"
                style="width: 120px"
                @pressEnter="save(record.id)"
              >
                <a-select-option value="user">普通用户</a-select-option>
                <a-select-option value="admin">管理员</a-select-option>
              </a-select>
              <check-outlined class="editable-cell-icon-check" @click="save(record.id)" />
            </div>
            <div v-else class="editable-cell-text-wrapper">
              <a-tag v-if="record.userRole === 'admin'" color="green">管理员</a-tag>
              <a-tag v-else color="blue">普通用户</a-tag>
              <edit-outlined class="editable-cell-icon" @click="edit(record.id)" />
            </div>
          </div>
        </template>
        <template v-else-if="column.dataIndex === 'userName'">
          <div class="editable-cell">
            <div v-if="editableData[record.id]" class="editable-cell-input-wrapper">
              <a-input
                v-model:value="editableData[record.id].userName"
                @pressEnter="save(record.id)"
              />
              <check-outlined class="editable-cell-icon-check" @click="save(record.id)" />
            </div>
            <div v-else class="editable-cell-text-wrapper">
              {{ record.userName || ' ' }}
              <edit-outlined class="editable-cell-icon" @click="edit(record.id)" />
            </div>
          </div>
        </template>
        <template v-else-if="column.dataIndex === 'userProfile'">
          <div class="editable-cell">
            <div v-if="editableData[record.id]" class="editable-cell-input-wrapper">
              <a-input
                v-model:value="editableData[record.id].userProfile"
                @pressEnter="save(record.id)"
              />
              <check-outlined class="editable-cell-icon-check" @click="save(record.id)" />
            </div>
            <div v-else class="editable-cell-text-wrapper">
              {{ record.userProfile || ' ' }}
              <edit-outlined class="editable-cell-icon" @click="edit(record.id)" />
            </div>
          </div>
        </template>
        <template v-if="column.dataIndex === 'createTime'">
          {{ dayjs(record.createTime).format('YYYY-MM-DD HH:mm:ss') }}
        </template>
        <template v-else-if="column.key === 'action'">
          <a-button danger @click="doDelete(record.id)">删除</a-button>
        </template>
      </template>
    </a-table>
  </div>
</template>
<script lang="ts" setup>
import { computed, onMounted, reactive, ref } from 'vue'
import {
  deleteUserUsingPost,
  listUserVoByPageUsingPost,
  updateUserUsingPost,
} from '@/api/userController.ts'
import { message, Modal } from 'ant-design-vue'
import dayjs from 'dayjs'
import type { UnwrapRef } from 'vue'
import {
  CheckOutlined,
  EditOutlined,
  CaretUpOutlined,
  CaretDownOutlined,
} from '@ant-design/icons-vue'
import { cloneDeep } from 'lodash-es'

const columns = [
  {
    title: 'id',
    dataIndex: 'id',
  },
  {
    title: '账号',
    dataIndex: 'userAccount',
  },
  {
    title: '用户名',
    dataIndex: 'userName',
  },
  {
    title: '头像',
    dataIndex: 'userAvatar',
  },
  {
    title: '简介',
    dataIndex: 'userProfile',
  },
  {
    title: '用户角色',
    dataIndex: 'userRole',
  },
  {
    title: '创建时间',
    dataIndex: 'createTime',
  },
  {
    title: '操作',
    key: 'action',
  },
]

// 定义数据
const dataList = ref<API.UserVO[]>([])
const total = ref(0)

// 可编辑数据
const editableData: UnwrapRef<Record<string, API.UserVO>> = reactive({})

// 搜索条件
const searchParams = reactive<API.UserQueryRequest>({
  current: 1,
  pageSize: 10,
  sortField: 'createTime',
  sortOrder: 'ascend',
})

// 获取数据
const fetchData = async () => {
  const res = await listUserVoByPageUsingPost({
    ...searchParams,
  })
  if (res.data.code === 0 && res.data.data) {
    dataList.value = res.data.data.records ?? []
    total.value = res.data.data.total ?? 0
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
}

// 页面加载时获取数据，请求一次
onMounted(() => {
  fetchData()
})

// 分页参数
const pagination = computed(() => {
  return {
    current: searchParams.current,
    pageSize: searchParams.pageSize,
    total: total.value,
    showSizeChanger: true,
    showTotal: (total: any) => `共 ${total} 条`,
    pageSizeOptions: ['10', '20', '50', '100'],
    showQuickJumper: false,
  }
})

// 表格变化之后，重新获取数据
const doTableChange = (page: any) => {
  searchParams.current = page.current
  searchParams.pageSize = page.pageSize
  fetchData()
}

// 搜索数据
const doSearch = () => {
  // 重置页码
  searchParams.current = 1
  fetchData()
}

// 删除数据
const doDelete = async (id: string) => {
  if (!id) {
    return
  }
  const res = await deleteUserUsingPost({ id })
  if (res.data.code === 0) {
    message.success('删除成功')
    // 刷新数据
    fetchData()
  } else {
    message.error('删除失败')
  }
}

// 编辑单元格
const edit = (id: string) => {
  editableData[id] = cloneDeep(dataList.value.filter((item) => item.id === id)[0])
}

// 保存编辑
const save = async (id: string) => {
  if (!editableData[id]) {
    return
  }

  // 获取原始数据
  const originalData = dataList.value.filter((item) => item.id === id)[0]
  const newData = editableData[id]

  // 检查角色是否发生变化
  const roleChanged = originalData.userRole !== newData.userRole

  if (roleChanged) {
    const oldRoleText = originalData.userRole === 'admin' ? '管理员' : '普通用户'
    const newRoleText = newData.userRole === 'admin' ? '管理员' : '普通用户'

    Modal.confirm({
      title: '确认角色修改',
      content: `确定将用户"${newData.userName}"的角色从"${oldRoleText}"修改为"${newRoleText}"吗？`,
      okText: '确认',
      cancelText: '取消',
      onOk: async () => {
        await performSave(id)
      },
      onCancel: () => {
        // 取消编辑，删除编辑数据
        delete editableData[id]
      },
    })
  } else {
    // 角色没有变化，直接保存
    await performSave(id)
  }
}

// 执行保存操作
const performSave = async (id: string) => {
  if (!editableData[id]) {
    return
  }

  try {
    const res = await updateUserUsingPost({
      id: editableData[id].id,
      userName: editableData[id].userName,
      userProfile: editableData[id].userProfile,
      userRole: editableData[id].userRole,
    })

    if (res.data.code === 0) {
      // 更新本地数据
      Object.assign(dataList.value.filter((item) => item.id === id)[0], editableData[id])
      delete editableData[id]
      message.success('更新成功')
    } else {
      message.error('更新失败：' + res.data.message)
    }
  } catch (error) {
    message.error('更新失败，请重试')
  }
}

// 切换排序
const toggleSort = () => {
  searchParams.sortOrder = searchParams.sortOrder === 'ascend' ? 'descend' : 'ascend'
  searchParams.current = 1 // 重置到第一页
  fetchData()
}
</script>

<style scoped>
.editable-cell {
  position: relative;
}

.editable-cell .editable-cell-input-wrapper,
.editable-cell .editable-cell-text-wrapper {
  padding-right: 24px;
}

.editable-cell .editable-cell-text-wrapper {
  padding: 5px 24px 5px 5px;
}

.editable-cell .editable-cell-icon,
.editable-cell .editable-cell-icon-check {
  position: absolute;
  right: 0;
  width: 20px;
  cursor: pointer;
}

.editable-cell .editable-cell-icon {
  margin-top: 4px;
  display: none;
}

.editable-cell .editable-cell-icon-check {
  line-height: 28px;
}

.editable-cell .editable-cell-icon:hover,
.editable-cell .editable-cell-icon-check:hover {
  color: #108ee9;
}

.editable-cell:hover .editable-cell-icon {
  display: inline-block;
}

.custom-header {
  display: flex;
  align-items: center;
  gap: 8px;
}

.sort-button {
  padding: 2px 4px;
  height: auto;
  min-height: auto;
}

.sort-button:hover {
  color: #1890ff;
}
</style>
