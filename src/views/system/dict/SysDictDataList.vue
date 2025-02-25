<template>
  <a-modal
    title="字典值管理"
    width="60%"
    :visible="visible"
    :footer="null"
    @cancel="handleCancel"
  >
    <a-card :bordered="false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="48">
            <a-col :md="8" :sm="24">
              <a-form-item label="字典名称">
                <a-input placeholder="请输入字典名称" v-model="queryParam.name" />
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <a-form-item label="字典值">
                <a-input placeholder="请输入字典值" v-model="queryParam.value" />
              </a-form-item>
            </a-col>
            <a-col :md="8" :sm="24">
              <span class="table-page-search-submitButtons">
                <a-button type="primary" @click="$refs.table.refresh(true)">查询</a-button>
                <a-button style="margin-left: 8px" @click="() => this.queryParam = {}">重置</a-button>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>

      <div class="table-operator">
        <a-button @click="handleAdd()" type="primary" icon="plus">新增
        </a-button>
        <a-dropdown v-if="selectedRowKeys.length > 0">
          <a-button type="danger" icon="delete" @click="delByIds(selectedRowKeys)">删除</a-button>
        </a-dropdown>
      </div>

      <s-table
        size="default"
        ref="table"
        rowKey="id"
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        :columns="columns"
        :data="loadData"
      >
        <span slot="status" slot-scope="text,record">
          <a-popconfirm placement="top" :title="text===true? '确定停用？':'确定启用？'" @confirm="() => editStatus(record)">
            <a>{{ statusFilter(text) }}</a>
          </a-popconfirm>
        </span>
        <span slot="action" slot-scope="text,record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical" />
          <a-popconfirm placement="topRight" title="确认删除？" @confirm="() => delByIds([[record.id]])">
            <a>删除</a>
          </a-popconfirm>
        </span>
      </s-table>
      <sysDictData-modal ref="modal" @ok="handleOk" />
    </a-card>
  </a-modal>
</template>

<script>
import { STable } from '@/components'
import { getSysDictDataPageList, delSysDictData, saveSysDictData } from '@/api/system/dict/sysDictData'
import SysDictDataModal from './modal/SysDictDataModal.vue'

export default {
  name: 'TableList',
  components: {
    STable,
    SysDictDataModal
  },
  data () {
    return {
      visible: false,
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },
      form: this.$form.createForm(this),
      mdl: {},
      statusDictTypeDropDown: [],
      // 查询参数
      queryParam: {},
      // 表头
      columns: [
        {
          title: '字典名称',
          dataIndex: 'name',
          scopedSlots: { customRender: 'name' },
          align: 'center'
        },
        {
          title: '字典值',
          dataIndex: 'value',
          scopedSlots: { customRender: 'value' },
          align: 'center'
        },
        {
          title: '状态',
          dataIndex: 'status',
          scopedSlots: { customRender: 'status' },
          align: 'center'
        },
        {
          title: '排序',
          dataIndex: 'sort',
          scopedSlots: { customRender: 'sort' },
          align: 'center'
        },
        {
          title: '操作',
          width: '200px',
          dataIndex: 'action',
          scopedSlots: { customRender: 'action' },
          align: 'center'
        }
      ],
      typeId: 0,
      endId: 0,
      // 加载数据方法 必须为 Promise 对象
      loadData: parameter => {
        return getSysDictDataPageList(Object.assign(parameter, this.queryParam)).then(res => {
          this.endId = res.data.totalRows !== 0 ? res.data.totalRows + 1 : 1
          return res.data
        })
      },
      selectedRowKeys: [],
      selectedRows: []
    }
  },
  methods: {
    // 打开此页面首先加载此方法
    index (record, statusDictTypeDropDown) {
      this.visible = true
      this.typeId = record.id
      this.queryParam.typeId = record.id
      this.statusDictTypeDropDown = statusDictTypeDropDown
      try {
        this.$refs.table.refresh()
      } catch (e) {
      }
    },
    onSelectChange (selectedRowKeys, selectedRows) {
      this.selectedRowKeys = selectedRowKeys
      this.selectedRows = selectedRows
    },
    statusFilter (status) {
      // eslint-disable-next-line eqeqeq
      const values = this.statusDictTypeDropDown.filter(item => item.key == status)
      if (values.length > 0) {
        return values[0].name
      }
    },
    editStatus (record) {
      saveSysDictData({ id: record.id, status: !record.status }).then(res => {
        if (res.success) {
          this.$message.success('操作成功')
          this.$refs.table.refresh()
        } else {
          this.$message.error('操作失败：' + res.message)
        }
      })
    },
    handleAdd () {
      this.$refs.modal.add(this.typeId, this.endId)
    },
    handleEdit (record) {
      this.$refs.modal.edit(record)
    },
    handleOk () {
      this.$refs.table.refresh(true)
    },
    handleCancel () {
      this.queryParam = {}
      this.visible = false
    },
    delByIds (ids) {
      delSysDictData({ id: ids.join(',') }).then(res => {
        if (res.code === 200) {
          this.$message.success('删除成功')
          this.handleOk()
        } else {
          this.$message.error(res.message)
        }
        this.selectedRowKeys = []
      })
    }
  }
}
</script>
