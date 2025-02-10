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
    this.loadTreeData()
  },
  methods: {
    fetchData (url) {
      return axios.get(url)
        .then(response => {
          // this.$log.push(JSON.stringify(response.data))
          // console.log('HTTP Response:', response.data) // 在这里打印返回的数据
          return response.data
        })
        .catch(error => {
          console.error('Error fetching data:', error)
          throw error
        })
    },
    async loadTreeData () {
      this.fetchData('http://127.0.0.1:1999/info')
        .then(data => {
          this.treeData = this.transformData(data)
          // console.log('transform data:', this.treeData) // 在这里打印返回的数据
        })
    },
    async handleNodeClick (data, node) {
      this.fetchData('http://127.0.0.1:1999/info')
        .then(data => {
          node.data.children = this.transformData(data)
        })
    },
    async handleNodeExpand (data, node) {
      // console.log('Expand node:', data.name)
      this.fetchData('http://127.0.0.1:1999/info')
        .then(data => {
          node.data.children = this.transformData(data)
        })
      // this.$refs.tree.updateKeyChildren(node.data.name, transformedData) // 更新树节点
      // console.log('updateKeyChildren:', node.data.name, data)
    },
    transformData (data) {
      // 假设 data 是一个字典结构，需要转换为数组结构
      const transform = (key, node) => {
        const typ = Object.prototype.toString.call(node)
        if (typ !== '[object Object]') {
          key = key + ': ' + node
          node = null
        }
        console.log('node:', key, node)
        return {
          name: key,
          value: node,
          children: node ? [{ name: '{...}', value: node }] : null
          // children: node.children ? node.children.map(transform) : null
        }
      }
      return Object.keys(data).map(key => transform(key, data[key]))
    }
  }
}
</script>
