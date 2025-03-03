<template>
  <d2-container>
    <el-tree ref="tree" :data="treeData" :props="defaultProps" node-key="name" @node-click="handleNodeClick" @node-expand="handleNodeExpand"></el-tree>
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
    this.addr = this.$route.params.addr || this.$route.query.addr
    if (!this.addr) {
      this.addr = '00000071'
      // console.error('未提供 addr 参数')
      // return
    }
    this.registry = this.$route.query.registry || 'service'
    this.loadInitalData(this.addr)
  },
  methods: {
    fetchData (keys) {
      const requestBody = {
        show: this.registry,
        val: keys
      }
      const config = {
        headers: {
          'Content-Type': 'application/json'
        }
      }
      return axios.post(`http://127.0.0.1:1999/debug?op=registry&node=game_1&addr=${this.addr}`,
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
      this.treeData = this.transformData(data.vals)
    },
    async handleNodeClick (data, node) {
      const childData = await this.fetchData(data.keys)
      node.data.children = this.transformData(childData.vals, data.keys)
    },
    transformData (data, parent_keys) {
      // 如果 data 是空的，直接返回空数组
      if (!data || Object.keys(data).length === 0) {
        return []
      }

      const transform = (idx, node) => {
        const expand = node[2] === 'table' || node[2] === 'function'
        const keys = parent_keys ? [...parent_keys, node[0]] : [node[0]]
        return {
          keys: keys,
          name: node[0] + ' :' + node[1],
          valuetype: node[2],
          children: expand ? [{ name: '{...}' }] : null
        }
      }
      return Object.keys(data).map(idx => transform(idx, data[idx]))
    }
  }
}
</script>
