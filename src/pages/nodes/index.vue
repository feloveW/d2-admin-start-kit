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
      columns: [
        // ...
      ]
    }
  },
  created () {
    // 从路由参数中获取 node
    this.server = this.$route.params.server || this.$route.query.server
    if (!this.server) {
      this.$message.error('错误：未提供 server 参数')
      return
    }
    this.loadInitalData()
  },
  methods: {
    fetchData () {
      console.log('开始发送请求...')
      return axios.get(`/${this.server}/get_clusters`)
        .then(response => {
          console.log('请求成功:', response)
          return response.data
        })
        .catch(error => {
          console.error('请求失败:', error)
          this.$message.error('请求失败，请检查网络连接')
          return []
        })
    },
    async loadInitalData () {
      const data = await this.fetchData()
      this.treeData = this.transformData(data)
    },
    transformData (data) {
      // 如果 data 是空的，直接返回空数组
      if (!data || Object.keys(data).length === 0) {
        return []
      }

      const transform = (idx, value) => {
        return {
          name: value,
          isLink: true,  // 添加标记表示这是可点击的链接
        }
      }
      return Object.keys(data).map(idx => transform(idx, data[idx]))
    },
    handleNodeClick(data) {
      if (data.isLink) {
        // 跳转到 list 页面，并传递必要的参数
        this.$router.push({
          path: '/list',
          query: {
            node: data.name,
            server: this.server
          }
        })
      }
    },
  }
}
</script>
