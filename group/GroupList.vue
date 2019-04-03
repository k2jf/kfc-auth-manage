<!-- 用户组列表 -->
<template>
  <div style="height: 100%;">
    <Row :gutter="16" class="margin-bottom">
      <Col span="14">
      <Button
        type="primary"
        @click="isShowGroupModal = true"
      >
        添加已有用户组
      </Button>
      </Col>
      <Col span="10">
      <Input
        placeholder="搜索用户组"
        v-model="group.filterName"
        @on-change="onSearchClick"></Input>
      </Col>
    </Row>
    <GroupEdit
      :currentRole="currentRole"
      :isShowGroupModal="isShowGroupModal"
      v-if="currentRole"
      @on-submit="reloadGroupList"
      @on-close="isShowGroupModal = false" />
    <Table
      :columns="group.columns"
      :data="group.data"
      size="small"
      :loading="group.loading"
      class="margin-bottom"></Table>
    <Page
      :total="group.total"
      class="page-container"
      @on-change="onPageChange"
    />
  </div>
</template>
<script>
// eslint-disable-next-line
import { Table, Page, Icon, Col, Row, Button, Input } from 'iview'
import GroupEdit from './GroupEdit.vue'

import { api } from '../api'

export default {
  name: 'GroupList',
  components: {
    Table,
    Page,
    Row,
    Col,
    Button,
    Input,
    GroupEdit
  },
  props: {
    currentRole: {
      type: Object,
      required: false,
      default: () => {
        return {}
      }
    }
  },
  data () {
    return {
      isShowGroupModal: false,
      group: {
        filterName: '',
        page: 1,
        size: 10,
        total: 0,
        loading: false,
        data: [],
        columns: [
          { title: '名称', key: 'name', minWidth: 110 },
          { title: '描述', key: 'email', minWidth: 130 },
          {
            title: '操作',
            width: 100,
            render: (h, params) => {
              return h(
                'span',
                {
                  style: {
                    cursor: 'pointer'
                  },
                  on: {
                    click: () => {
                      this.deleteGroup(params.row.name)
                    }
                  }
                },
                [
                  h(
                    'Icon',
                    {
                      props: {
                        type: 'ios-trash',
                        size: 16,
                        color: '#5cadff'
                      }
                    }
                  ),
                  h(
                    'span', '移除'
                  )
                ]
              )
            }
          }
        ]
      }
    }
  },
  watch: {
    currentRole: {
      handler (curVal, oldVal) {
        if (curVal && curVal.id) {
          this.getGroupList()
        }
      }
    }
  },
  mounted () {
    if (this.currentRole) {
      this.getGroupList()
    }
  },
  methods: {
    // 删除用户组
    deleteGroup (name) {
      this.$axios.put(`${api.roles}/${this.currentRole.id}/remove`, { users: [{ name: name }] }).then(res => {
        this.getGroupList()
      })
    },
    // 获取用户组列表
    getGroupList () {
      this.group.loading = true
      let { page, size } = this.group
      let name = this.currentRole.name

      if (this.group.filterName) {
        // url + 过滤条件
      }
      this.$axios.get(`${api.users}/?page=${page}&size=${size}&rolename=${name}`).then(res => {
        this.group.data = res.data.result
        this.group.total = res.data.pages.total
        this.group.loading = false
      })
    },
    // 切换页数
    onPageChange (page) {
      this.group.page = page
      this.getGroupList()
    },
    onSearchClick () {
      this.getGroupList()
    },
    reloadGroupList () {
      this.isShowGroupModal = false
      if (this.currentRole) {
        // 添加用户组后刷新用户组列表页面
        this.getGroupList()
      }
    }
  }
}
</script>
