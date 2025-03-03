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
      }
    }
  },
  created () {
    // 从路由参数中获取 addr
    this.node = this.$route.params.node || this.$route.query.node
    if (!this.node) {
      this.node = 'game_1'
      // console.error('未提供 addr 参数')
      // return
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
      return axios.post(`http://127.0.0.1:1999/debug?op=list&node=${this.node}`,
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
              url: `http://127.0.0.1:1999/debug?op=registry&node=${this.node}&addr=${node.addr}`
            },
            {
              name: 'service',
              isLink: true,  // 添加标记，用于识别可点击项
              addr: node.addr,  // 保存addr用于跳转
              url: `http://127.0.0.1:1999/debug?op=service&node=${this.node}&addr=${node.addr}`
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
            registry: data.name
          }
        })
      }
    }
  }
}
</script>
