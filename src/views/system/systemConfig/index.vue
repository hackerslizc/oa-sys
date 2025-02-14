<template>
  <div class="p-4">
    <div class="mb-3">
      <form-search
        :formFields="formFields"
        @search="handleQuery"
        @reset="handleQuery"
      />
    </div>
    <a-row :gutter="10" class="mb-2">
      <a-col v-has-permi="['system:systemConfig:add']">
        <a-button color="success" @click="handleAdd">{{
          t('common.add')
        }}</a-button>
      </a-col>
      <a-col v-has-permi="['system:systemConfig:delete']">
        <a-popconfirm
          :title="t('common.confirmDelete')"
          :ok-text="t('common.okText')"
          :cancel-text="t('common.cancelText')"
          @confirm="confirm"
          @cancel="cancel"
        >
          <a-button :disabled="!hasSelected" color="error">
            {{ t('common.update') }}
          </a-button>
        </a-popconfirm>
      </a-col>
    </a-row>

    <a-table
      :loading="loading"
      rowKey="id"
      :row-selection="{
        selectedRowKeys: selectedRowKeys,
        onChange: onSelectChange,
      }"
      :columns="columns"
      :data-source="systemConfigList"
      :pagination="pagination"
      @change="handleTableChange"
    >
      <template #action="{ record }">
        <span>
          <a-button
            type="link"
            color="success"
            class="mr-3"
            @click="handleUpdate(record)"
            v-has-permi="['system:systemConfig:update']"
          >
            {{ t('common.update') }}
          </a-button>
          <a-popconfirm
            :title="t('common.confirmDelete')"
            :ok-text="t('common.okText')"
            :cancel-text="t('common.cancelText')"
            @confirm="confirm(record)"
            @cancel="cancel"
          >
            <a-button
              type="link"
              color="error"
              v-has-permi="['system:systemConfig:delete']"
            >
              {{ t('common.delete') }}
            </a-button>
          </a-popconfirm>
        </span>
      </template>
    </a-table>

    <!-- 新增修改推窗 -->
    <a-drawer
      width="600px"
      :title="drawerTitle"
      placement="right"
      v-model:visible="open"
      :maskClosable="false"
      @close="handleClose"
    >
      <a-form
        v-if="open"
        ref="formRef"
        :model="formState"
        :rules="rules"
        :label-col="labelCol"
        :wrapper-col="wrapperCol"
      >
        <a-row>
          <a-col :span="24">
            <a-form-item :label="t('routes.systemConfig.name')" name="name">
              <a-input
                v-model:value="formState.name"
                :placeholder="t('routes.systemConfig.namePlaceholder')"
              />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item
              :label="t('routes.systemConfig.keyName')"
              name="keyName"
            >
              <a-input
                v-model:value="formState.keyName"
                :placeholder="t('routes.systemConfig.keyNamePlaceholder')"
              />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item :label="t('routes.systemConfig.key')" name="key">
              <a-input
                v-model:value="formState.key"
                :placeholder="t('routes.systemConfig.keyPlaceholder')"
              />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item :label="t('routes.systemConfig.remark')" name="remark">
              <a-textarea
                :rows="3"
                v-model:value="formState.remark"
                :placeholder="t('routes.systemConfig.remarkPlaceholder')"
              />
            </a-form-item>
          </a-col>
          <a-col :span="24">
            <a-form-item>
              <a-button type="primary" class="mr-3" @click="handleSubmit">
                {{ t('common.okText') }}
              </a-button>
              <a-button @click="handleClose">
                {{ t('common.cancelText') }}
              </a-button>
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </a-drawer>
  </div>
</template>
<script lang="ts">
import {
  defineComponent,
  reactive,
  ref,
  onMounted,
  nextTick,
  toRefs,
  computed,
} from 'vue'
import {
  getSystemConfig,
  getSystemConfigById,
  delSystemConfig,
  addSystemConfig,
  updateSystemConfig,
} from '@/api/admin/system/systemConfig'
import { selectDictLabel } from '@/utils/dictFormat'
import useDrawer from '@/hooks/useDrawer'
import { message as Message } from 'ant-design-vue'
import { ValidateErrorEntity } from 'ant-design-vue/es/form/interface'
import { useAppStore } from '@/store/modules/app'
import { TableState } from 'ant-design-vue/es/table/interface'

import FormSearch from '@/components/FormSearch/index.vue'
import { ISystemConfig } from '@/api/admin/system/systemConfig/type'
import { useI18n } from '@/hooks/useI18n'

const { t } = useI18n()

interface FormState {
  id: undefined | number
  name: undefined | string
  keyName: undefined | number
  key: undefined | string
  status: undefined | string
  remark?: undefined | string
  [key: string]: any
}
type Pagination = TableState['pagination']

const columns = [
  {
    title: t('routes.systemConfig.name'),
    dataIndex: 'name',
    key: 'name',
    align: 'center',
  },
  {
    title: t('routes.systemConfig.keyName'),
    dataIndex: 'keyName',
    key: 'keyName',
    align: 'center',
  },
  {
    title: t('routes.systemConfig.key'),
    dataIndex: 'key',
    key: 'key',
    align: 'center',
  },
  {
    title: t('routes.systemConfig.remark'),
    dataIndex: 'remark',
    key: 'remark',
    align: 'center',
  },
  {
    title: t('routes.systemConfig.createdBy'),
    dataIndex: 'createdBy',
    key: 'createdBy',
    align: 'center',
  },
  {
    title: t('routes.systemConfig.createdAt'),
    dataIndex: 'createdAt',
    key: 'createdAt',
    align: 'center',
  },
  {
    title: t('routes.systemConfig.action'),
    key: 'action',
    align: 'center',
    slots: { customRender: 'action' },
  },
]

export default defineComponent({
  components: {
    FormSearch,
  },
  setup() {
    const loading = computed(() => useAppStore().loading)
    const rules = {
      name: [
        {
          required: true,
          message: t('routes.systemConfig.nameCannotBeEmpty'),
          trigger: 'blur',
        },
      ],
      keyName: [
        {
          required: true,
          message: t('routes.systemConfig.keyNameCannotBeEmpty'),
          trigger: 'blur',
        },
      ],
      key: [
        {
          required: true,
          message: t('routes.systemConfig.keyCannotBeEmpty'),
          trigger: 'change',
        },
      ],
    }
    const formFields = reactive([
      {
        type: 'input',
        label: t('routes.systemConfig.name'),
        name: 'name',
        value: '',
        placeholder: t('routes.systemConfig.namePlaceholder'),
      },
    ])
    // 查询表单操作
    const queryParams = reactive({
      pageNum: 1,
      pageSize: 10,
      name: undefined || '',
      keyName: undefined || '',
      status: undefined || '',
    })

    const handleQuery = (query: {
      name: string
      keyName: string
      status: string
    }) => {
      pagination.value.current = 1
      queryParams.pageNum = pagination.value.current
      queryParams.name = query.name
      queryParams.keyName = query.keyName
      queryParams.status = query.status
      getList(queryParams)
    }
    // 表格操作
    const systemConfigList = ref<ISystemConfig[]>([])
    const pagination = ref({
      total: 0,
      current: 1,
      pageSize: 10,
      showSizeChanger: true,
      showTotal: (total) => `共 ${total} 条`,
    })
    const handleTableChange = (page: Pagination) => {
      (pagination.value as Pagination) = page
      queryParams.pageNum = pagination.value.current
      queryParams.pageSize = pagination.value.pageSize
      getList(queryParams)
    }
    const state = reactive({
      selectedRowKeys: [],
    })
    const hasSelected = computed(() => state.selectedRowKeys.length > 0)

    const onSelectChange = (selectedRowKeys) => {
      console.log('selectedRowKeys changed: ', selectedRowKeys)
      state.selectedRowKeys = selectedRowKeys
    }

    const getList = (queryParams?: {}) => {
      getSystemConfig(queryParams).then((res) => {
        console.log(res)
        systemConfigList.value = res.data.rows
        pagination.value.total = res.data.count
        state.selectedRowKeys = []
      })
    }

    const init = () => {
      getList(queryParams)
    }

    const formRef = ref()
    const formState: FormState = reactive({
      id: undefined,
      name: undefined,
      keyName: undefined,
      status: '1',
      key: undefined,
    })
    const { open, drawerTitle } = useDrawer()
    console.log(open)
    const handleClose = () => {
      formState.id = undefined
      formRef.value.resetFields()
      console.log(formRef)
      open.value = false
    }
    // 表单提交
    const handleSubmit = () => {
      console.log(formState)
      formRef.value
        .validate()
        .then(() => {
          if (formState.id) {
            updateSystemConfig(formState).then((res) => {
              Message.success(res.message)
              getList(queryParams)
              formState.id = undefined
              formRef.value.resetFields()
              open.value = false
            })
          } else {
            addSystemConfig(formState).then((res) => {
              Message.success(res.message)
              getList(queryParams)
              formState.id = undefined
              formRef.value.resetFields()
              open.value = false
            })
          }
        })
        .catch((error: ValidateErrorEntity) => {
          console.log('error', error)
        })
    }
    // 确认删除
    const confirm = (row) => {
      const ids = row.id || state.selectedRowKeys
      delSystemConfig(ids).then(() => {
        if (
          (ids.length && ids.length === systemConfigList.value.length) ||
          systemConfigList.value.length === 1
        ) {
          if (
            Math.ceil(pagination.value.total / queryParams.pageSize) ===
              queryParams.pageNum &&
            queryParams.pageNum > 1
          ) {
            queryParams.pageNum--
          }
        }
        getList(queryParams)
        Message.success(t('common.deleteSuccess'))
      })
    }
    // 取消删除
    const cancel = (e: MouseEvent) => {
      console.log(e)
      Message.success(t('common.cancelDelete'))
    }

    // 新增按钮操作
    const handleAdd = () => {
      open.value = true
      drawerTitle.value = t('common.add')
    }
    // 更新按钮操作
    const handleUpdate = (row) => {
      getSystemConfigById(row.id).then((res) => {
        open.value = true
        drawerTitle.value = t('common.update')
        nextTick(() => {
          Object.keys(formState).forEach((key) => {
            formState[key] = res.data[key]
          })
        })
      })
    }

    onMounted(async () => {
      init()
    })

    return {
      t,
      loading,
      queryParams,
      formFields,
      handleQuery,
      systemConfigList,
      columns,
      pagination,
      handleTableChange,
      selectDictLabel,
      ...toRefs(state),
      hasSelected,
      onSelectChange,

      open,
      drawerTitle,
      formState,
      labelCol: { span: 4 },
      wrapperCol: { span: 18 },
      rules,
      formRef,
      handleClose,
      handleSubmit,
      confirm,
      cancel,
      handleAdd,
      handleUpdate,
    }
  },
})
</script>
