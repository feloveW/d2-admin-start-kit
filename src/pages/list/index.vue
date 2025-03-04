<template>
  <d2-container>
    <el-tree ref="tree" :data="treeData" :props="defaultProps" node-key="name" @node-click="handleNodeClick"></el-tree>
  </d2-container>
</template>

<script>
import axios from 'axios'

export default {
  name: 'DynamicTreeView',
  data () {
    return {
      treeData: [],
      defaultProps: {
        children: 'children', // 假设返回的数据中子节点被命名为'children'
        label: 'name' // 假设每个节点的显示名称为'name'
      },
      addr: ''
    }
  },
  created () {
    // 从路由参数中获取 node
    this.server = this.$route.params.server || this.$route.query.server
    if (!this.server) {
      this.$message.error('错误：未提供 server 参数')
      return
    }
    this.node = this.$route.params.node || this.$route.query.node
    if (!this.node) {
      this.$message.error('错误：未提供 node 参数')
      return
    }
    this.loadInitalData(this.node)
  },
  methods: {
    fetchData (keys) {
      const requestBody = {
      }
      const config = {
        headers: {
          'Content-Type': 'application/json'
        }
      }
      return axios.post(`/${this.server}/debug?op=list&node=${this.node}`,
        requestBody,
        config
      )
      .then(response => {
        return response.data
        // console.log('transform data:', this.treeData) // 在这里打印返回的数据
      })

    },
    async loadInitalData () {
      const data = await this.fetchData()
      //console.log('data.list:', data.list)
      this.treeData = this.transformData(data.list)
    },
    transformData (data) {
      // 如果 data 是空的，直接返回空数组
      if (!data || Object.keys(data).length === 0) {
        return []
      }

      const transform = (idx, node) => {
        return {
          name: node.name + ' : ' + node.addr,
          registry: node.name,
          addr: node.addr,
          children: [
            {
              name: 'registry',
              isLink: true,  // 添加标记，用于识别可点击项
              addr: node.addr,  // 保存addr用于跳转
            },
            {
              name: 'service',
              isLink: true,  // 添加标记，用于识别可点击项
              addr: node.addr,  // 保存addr用于跳转
            }
          ]
        }
      }
      return Object.keys(data).map(idx => transform(idx, data[idx]))
    },
    handleNodeClick(data) {
      if (data.isLink) {
        // 使用 Vue Router 跳转到 info 页面
        this.$router.push({
          path: '/info',
          query: {
            addr: data.addr,
            registry: data.name,
            server: this.server,
            node: this.node
          }
        })
      }
    }
  }
}
</script>
