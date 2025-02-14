<template>
  <div class="p-4">
    <a-row :gutter="15">
      <a-col :span="4">
        <dept-search
          ref="deptSearchRef"
          :original="originalTree"
          :data="deptTreeOption"
          :baseFields="replaceFields"
          @select="handleSelect"
        />
      </a-col>
      <a-col :span="20">
        <div class="mb-3">
          <form-search
            :formFields="formFields"
            @search="handleQuery"
            @reset="handleQuery"
          />
        </div>
        <a-row :gutter="10" class="mb-2">
          <a-col v-has-permi="['system:user:add']">
            <a-button color="success" @click="handleAdd">
              {{ t('common.add') }}
            </a-button>
          </a-col>
          <a-col v-has-permi="['system:user:delete']">
            <a-popconfirm
              :title="t('common.confirmDelete')"
              :ok-text="t('common.okText')"
              :cancel-text="t('common.cancelText')"
              @confirm="confirm"
              @cancel="cancel"
            >
              <a-button :disabled="!hasSelected" color="error">
                {{ t('common.delete') }}
              </a-button>
            </a-popconfirm>
          </a-col>
          <!-- <a-col>
            <a-button color="warning">
              {{ t('common.export') }}
            </a-button>
          </a-col> -->
          <!-- <a-col>
            <a-button color="normal">
              {{ t('common.import') }}
            </a-button>
          </a-col> -->
        </a-row>

        <a-table
          :loading="loading"
          rowKey="id"
          :row-selection="{
            selectedRowKeys: selectedRowKeys,
            onChange: onSelectChange,
          }"
          :columns="columns"
          :data-source="userList"
          :pagination="pagination"
          @change="handleTableChange"
        >
          <template #department="{ record }">
            <span>
              {{ record.department.deptName }}
            </span>
          </template>
          <template #status="{ record }">
            <span>{{ selectDictLabel(statusOptions, record.status) }}</span>
          </template>
          <template #roles="{ record }">
            <span>{{ getRoles(record) }}</span>
          </template>
          <template #action="{ record }">
            <span>
              <a-button
                type="link"
                color="success"
                class="mr-3"
                @click="handleUpdate(record)"
                v-has-permi="['system:user:update']"
              >
                {{ t('common.update') }}
              </a-button>
              <a-button
                type="link"
                color="success"
                class="mr-3"
                @click="showModal(record)"
                v-has-permi="['system:user:resetPwd']"
              >
                {{ t('routes.user.resetPwd') }}
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
                  v-has-permi="['system:user:delete']"
                >
                  {{ t('common.delete') }}
                </a-button>
              </a-popconfirm>
            </span>
          </template>
        </a-table>
      </a-col>
    </a-row>

    <!-- 新增修改用户 -->
    <a-drawer
      width="50%"
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
          <a-col :span="12" v-show="formState.deptId !== 0">
            <a-form-item :label="t('routes.user.deptId')" name="deptId">
              <treeselect
                ref="treeRef"
                class="!mt-[3px]"
                v-model:value="formState.deptId"
                :normalizer="normalizer"
                :placeholder="t('routes.user.deptIdPlaceholder')"
                :options="deptOptions"
                @select="handleTreeSelect"
              />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item :label="t('routes.user.nickName')" name="nickName">
              <a-input
                v-model:value="formState.nickName"
                :placeholder="t('routes.user.nickNamePlaceholder')"
              />
            </a-form-item>
          </a-col>
          <a-col :span="12" v-if="!isUpdate">
            <a-form-item :label="t('routes.user.userName')" name="userName">
              <a-input
                v-model:value="formState.userName"
                :placeholder="t('routes.user.userNamePlaceholder')"
              />
            </a-form-item>
          </a-col>
          <a-col :span="12" v-if="!isUpdate">
            <a-form-item :label="t('routes.user.password')" name="password">
              <a-input-password
                v-model:value="formState.password"
                :placeholder="t('routes.user.passwordPlaceholder')"
              />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item :label="t('routes.user.roleIds')" name="roleIds">
              <a-select
                mode="multiple"
                v-model:value="formState.roleIds"
                :placeholder="t('routes.user.roleIdsPlaceholder')"
              >
                <a-select-option v-for="item in roleOptions" :key="item.id">
                  {{ item.roleName }}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item :label="t('routes.user.sex')" name="sex">
              <a-radio-group
                v-model:value="formState.sex"
                :options="sexOptions"
              />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item :label="t('routes.user.mobile')" name="mobile">
              <a-input
                v-model:value="formState.mobile"
                :placeholder="t('routes.user.mobilePlaceholder')"
              />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item :label="t('routes.user.status')" name="status">
              <a-radio-group
                v-model:value="formState.status"
                :options="disableOptions"
              />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item :label="t('routes.user.email')" name="email">
              <a-input
                v-model:value="formState.email"
                :placeholder="t('routes.user.emailPlaceholder')"
              />
            </a-form-item>
          </a-col>
          <a-col :span="12">
            <a-form-item :label="t('routes.user.remark')" name="remark">
              <a-textarea
                :rows="3"
                v-model:value="formState.remark"
                :placeholder="t('routes.user.remarkPlaceholder')"
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

    <!-- 重置密码 -->
    <a-modal
      v-model:visible="visible"
      :title="t('routes.user.resetPwd')"
      @ok="handleResetPwd"
      @cancel="handleResetClose"
    >
      <div class="flex items-center">
        <span class="whitespace-nowrap"
        >{{ t('routes.user.newPassword') }}：</span
        >
        <a-input-password
          v-model:value="newPassword"
          :placeholder="t('routes.user.newPasswordPlaceholder')"
        />
      </div>
    </a-modal>
  </div>
</template>
<script lang="ts">
import {
  defineComponent,
  reactive,
  ref,
  onMounted,
  toRefs,
  computed,
  nextTick,
} from 'vue'
import { TreeDataItem } from 'ant-design-vue/es/tree/Tree'
import { ValidateErrorEntity } from 'ant-design-vue/es/form/interface'
import { getDict, selectDictLabel } from '@/utils/dictFormat'
import { message as Message } from 'ant-design-vue'
import { getDept } from '@/api/admin/system/dept'
import {
  listUser,
  getUser,
  delUser,
  addUser,
  updateUser,
  // exportUser,
  resetUserPwd,
  // importTemplate,
} from '@/api/admin/system/user'
import { getRole } from '@/api/admin/system/role'
import { handleTree } from '@/utils/tools'
import useDrawer from '@/hooks/useDrawer'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import { useUserStore } from '@/store/modules/user'
import { formRules } from '@/utils/validate'
import { useAppStore } from '@/store/modules/app'
import { TableState } from 'ant-design-vue/es/table/interface'

// 组件
import DeptSearch from '@/components/DeptSearch/index.vue'
import FormSearch from '@/components/FormSearch/index.vue'
import { IUser } from '@/api/admin/system/user/type'
import { IRole } from '@/api/admin/system/role/type'
import { IData } from '@/api/admin/system/dict/data/type'
import { IDept } from '@/api/admin/system/dept/type'
import { useI18n } from '@/hooks/useI18n'

const { t } = useI18n()

interface FormState {
  id: undefined | number
  deptId: undefined | number
  nickName: undefined | string
  userName: undefined | string
  password: undefined | string
  sex: undefined | string
  roleIds: undefined | number[]
  mobile: undefined | number | string
  status: undefined | string
  email: undefined | string
  remark: undefined | string
}

type Pagination = TableState['pagination']

const columns = [
  {
    title: t('routes.user.userName'),
    dataIndex: 'userName',
    key: 'userName',
    align: 'center',
  },
  {
    title: t('routes.user.nickName'),
    dataIndex: 'nickName',
    key: 'nickName',
    align: 'center',
  },
  {
    title: t('routes.user.roleIds'),
    key: 'roles',
    align: 'center',
    slots: {
      customRender: 'roles',
    },
  },
  {
    title: t('routes.user.department'),
    key: 'department',
    align: 'center',
    slots: {
      customRender: 'department',
    },
  },
  {
    title: t('routes.user.mobile'),
    key: 'mobile',
    dataIndex: 'mobile',
    align: 'center',
  },
  {
    title: t('routes.user.status'),
    dataIndex: 'status',
    key: 'status',
    align: 'center',
    slots: { customRender: 'status' },
  },
  {
    title: t('routes.user.createdAt'),
    dataIndex: 'createdAt',
    key: 'createdAt',
    align: 'center',
  },
  {
    title: t('routes.user.action'),
    key: 'action',
    align: 'center',
    slots: { customRender: 'action' },
  },
]

export default defineComponent({
  components: {
    Treeselect,
    DeptSearch,
    FormSearch,
  },
  setup() {
    const userStore = useUserStore()
    const loading = computed(() => useAppStore().loading)
    const roleOptions = ref<IRole[]>([])
    const statusOptions = ref<IData[]>([])
    const positionOptions = ref<IData[]>([])
    const educationOptions = ref<IData[]>([])
    const disableOptions = ref<IData[]>([])
    const sexOptions = ref<IData[]>([])
    /**
     * 左侧树形控件操作
     */
    const deptSearchRef = ref()
    const originalTree = ref<IDept[]>([])
    const replaceFields = {
      children: 'children',
      title: 'deptName',
      key: 'deptId',
    }
    const deptTreeOption = ref<TreeDataItem[]>([])
    const handleSelect = (keys: number[]) => {
      queryParams.deptId = keys[0]
      handleQuery(queryParams)
    }

    // 查询表单操作
    const queryRef = ref()
    const queryParams = reactive({
      userName: undefined || '',
      deptId: 0,
      roleId: undefined || '',
      pageNum: 1,
      pageSize: 10,
    })
    const formFields = reactive([
      {
        type: 'input',
        label: t('routes.user.userName'),
        name: 'userName',
        value: undefined,
        placeholder: t('routes.user.userNamePlaceholder'),
      },
      {
        type: 'select',
        label: t('routes.user.roleIds'),
        name: 'roleId',
        value: undefined,
        placeholder: t('routes.user.roleIdsPlaceholder'),
        normalizer: {
          value: 'id',
          label: 'roleName',
        },
        options: roleOptions,
      },
    ])

    const handleQuery = (query: { userName: string; roleId: string }) => {
      pagination.value.current = 1
      queryParams.pageNum = pagination.value.current
      queryParams.userName = query.userName
      queryParams.roleId = query.roleId
      getList(queryParams)
    }

    /**
     * 表格操作
     */
    const state = reactive({
      selectedRowKeys: [],
    })
    const userList = ref<IUser[]>([])
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
    // 判断删除按钮是否可点击
    const hasSelected = computed(() => state.selectedRowKeys.length > 0)
    // 多选框选择操作
    const onSelectChange = (selectedRowKeys) => {
      state.selectedRowKeys = selectedRowKeys
    }

    // 格式角色
    const getRoles = (record) => {
      return record.roles.map((list) => list.roleName).join(',')
    }

    // 新增按钮操作
    const handleAdd = () => {
      open.value = true
      isUpdate.value = false
      drawerTitle.value = t('common.add')
    }

    // 更新按钮操作
    const handleUpdate = (row) => {
      getUser(row.id).then((res) => {
        open.value = true
        isUpdate.value = true
        drawerTitle.value = t('common.update')
        nextTick(() => {
          Object.keys(formState).forEach((key) => {
            formState[key] = res.data[key]
          })
          formState.roleIds = res.data.roles.map((list) => list.id)
          treeRef.value.forest.selectedNodeIds.push(res.data.deptId)
          console.log(formState)
        })
      })
    }

    // 确认删除
    const confirm = (row) => {
      const ids = row.id || state.selectedRowKeys
      console.log(ids)
      delUser(ids).then(() => {
        if (
          (ids.length && ids.length === userList.value.length) ||
          userList.value.length === 1
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
    // 部门树选项
    const deptOptions = ref<IDept[]>([])

    // 获取表格数据
    const getList = (queryParams?: {}) => {
      listUser(queryParams).then((res) => {
        userList.value = res.data.rows
        pagination.value.total = res.data.count
      })

      getDept().then((res) => {
        // 获取菜单树
        const children = handleTree(
          res.data.rows,
          'deptId',
          'parentId',
          'children',
          userStore.userInfo.deptId
        ).tree
        const parent = res.data.rows.filter(
          (item) => item.deptId === userStore.userInfo.deptId
        )
        parent[0].children = children
        deptOptions.value = parent
        console.log(deptOptions)
      })
    }

    /**
     * 推窗表单操作
     */
    // 获取部门树
    const treeRef = ref()
    const formState: FormState = reactive({
      id: undefined,
      deptId: undefined,
      nickName: undefined,
      userName: undefined,
      password: undefined,
      sex: '1',
      roleIds: undefined || [],
      mobile: undefined,
      status: '1',
      email: undefined,
      remark: undefined,
    })
    const rules = {
      deptId: [
        {
          required: true,
          validator: formRules.number,
          message: t('routes.user.deptIdCannotBeEmpty'),
          trigger: 'change',
        },
      ],
      nickName: [
        {
          required: true,
          message: t('routes.user.nickNameCannotBeEmpty'),
          trigger: 'blur',
        },
      ],
      userName: [
        {
          required: true,
          message: t('routes.user.userNameCannotBeEmpty'),
          trigger: 'blur',
        },
      ],
      password: [
        {
          required: true,
          message: t('routes.user.passwordCannotBeEmpty'),
          trigger: 'blur',
        },
      ],
      roleIds: [
        {
          required: true,
          validator: formRules.checkRoleLength,
          trigger: 'change',
        },
      ],
    }
    const { open, drawerTitle } = useDrawer()
    // 是否更新操作
    const isUpdate = ref(false)
    // 上级部门选中事件
    const handleTreeSelect = (node) => {
      formState.deptId = node.deptId
    }

    // 新增修改表单操作
    const formRef = ref()

    // 表单提交
    const handleSubmit = () => {
      console.log(formState)
      formRef.value
        .validate()
        .then(() => {
          if (formState.id) {
            updateUser(formState).then((res) => {
              Message.success(res.message)
              formState.id = undefined
              formState.deptId = undefined
              formState.password = undefined
              formState.userName = undefined
              formState.roleIds = undefined
              formRef.value.resetFields()
              open.value = false
              getList(queryParams)
            })
          } else {
            addUser(formState).then((res) => {
              Message.success(res.message)
              formState.id = undefined
              formState.deptId = undefined
              formState.password = undefined
              formState.userName = undefined
              formState.roleIds = undefined
              formRef.value.resetFields()
              open.value = false
              getList(queryParams)
            })
          }
        })
        .catch((error: ValidateErrorEntity) => {
          console.log('error', error)
        })
    }

    // 序列化部门
    const normalizer = (node) => {
      if (node.children && !node.children.length) {
        delete node.children
      }
      return {
        id: node.deptId,
        label: node.deptName,
        children: node.children,
      }
    }

    // 关闭推窗
    const handleClose = () => {
      formState.id = undefined
      formState.deptId = undefined
      formState.password = undefined
      formState.userName = undefined
      formRef.value.resetFields()
      console.log(formState)
      open.value = false
    }

    /**
     * 重置密码操作
     */
    const visible = ref<boolean>(false)
    const resetformState = reactive({
      id: 0,
      newPassword: '',
    })
    const showModal = (row) => {
      resetformState.id = row.id
      visible.value = true
    }
    // 重置密码
    const handleResetPwd = () => {
      if (!resetformState.newPassword) {
        Message.error(t('routes.user.newPasswordCannotBeEmpty'))
        return
      }
      resetUserPwd(resetformState.id, {
        newPassword: resetformState.newPassword,
      }).then((res) => {
        Message.success(res.message)
        resetformState.newPassword = ''
        visible.value = false
      })
    }
    // 取消重置密码
    const handleResetClose = () => {
      resetformState.newPassword = ''
      visible.value = false
    }

    // 获取部门数据
    const getDeptList = () => {
      getDept().then((res) => {
        originalTree.value = res.data.rows
        deptTreeOption.value = handleTree(
          res.data.rows,
          'deptId',
          'parentId',
          'children',
          0
        ).tree
        deptSearchRef.value.expandedKeys.value = originalTree.value.map(
          (item: TreeDataItem) => item.id
        )
      })
    }

    const init = () => {
      getDeptList()
      getList(queryParams)
      getRole().then((res) => {
        roleOptions.value = res.data.rows
      })
    }

    onMounted(async () => {
      statusOptions.value = await getDict('sys_normal_disable')
      positionOptions.value = await getDict('sys_user_position')
      educationOptions.value = await getDict('sys_user_education')
      disableOptions.value = await getDict('sys_normal_disable')
      sexOptions.value = await getDict('sys_user_sex')
      disableOptions.value.forEach((item) => {
        item.label = item.dictLabel
        item.value = item.dictValue
      })
      sexOptions.value.forEach((item) => {
        item.label = item.dictLabel
        item.value = item.dictValue
      })
      init()
    })

    return {
      t,
      loading,
      replaceFields,
      deptSearchRef,
      originalTree,
      deptTreeOption,
      handleSelect,

      queryRef,
      queryParams,
      formFields,
      handleQuery,

      statusOptions,
      columns,
      ...toRefs(state),
      userList,
      pagination,
      handleTableChange,
      selectDictLabel,
      hasSelected,
      onSelectChange,
      getRoles,
      confirm,
      cancel,
      handleAdd,
      handleUpdate,

      treeRef,
      formRef,
      rules,
      isUpdate,
      open,
      drawerTitle,
      deptOptions,
      normalizer,
      handleClose,
      handleSubmit,
      labelCol: { span: 6 },
      wrapperCol: { span: 18 },
      formState,
      roleOptions,
      handleTreeSelect,
      positionOptions,
      educationOptions,
      disableOptions,
      sexOptions,

      visible,
      ...toRefs(resetformState),
      handleResetPwd,
      handleResetClose,
      showModal,
    }
  },
})
</script>
