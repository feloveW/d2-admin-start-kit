<template>
  <d2-container>
    <el-tree ref="tree" :data="treeData" :props="defaultProps" node-key="name" @node-click="handleNodeClick"></el-tree>
  </d2-container>
</template>

<script>
export default {
  name: 'DynamicTreeView',
  data () {
    return {
      treeData: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      localConfig: {
        'wfl-251': {
          name: 'wfl-251',
        },
        'bwx-250': {
          name: 'bwx-250',
        },
        'js-33': {
          name: 'js-33',
        },
        'bwx-wan': {
          name: 'bwx-wan',
        },
        'bwx-fuyao': {
          name: 'bwx-fuyao',
        }
      }
    }
  },
  created () {
    // 直接使用本地配置初始化树
    this.initializeFromLocalConfig()
  },
  methods: {
    initializeFromLocalConfig() {
      this.treeData = this.transformData(this.localConfig)
    },
    transformData (data) {
      // 如果 data 是空的，直接返回空数组
      if (!data || Object.keys(data).length === 0) {
        return []
      }

      const transform = (key, value) => {
        return {
          name: value.name,
          isLink: true,
        }
      }
      return Object.keys(data).map(idx => transform(idx, data[idx]))
    },
    handleNodeClick(data) {
      if (data.isLink) {
        // 跳转到 nodes 页面
        this.$router.push({
          path: '/nodes',
          query: {
            server: data.name
          }
        })
      }
    }
  }
}
</script>
