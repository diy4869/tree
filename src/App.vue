<template>
  <div id="app">
    <ul>
      <li>
        <!-- <el-tree :data="data" show-checkbox></el-tree> -->
      </li>
      <li>
        <tree :data="data" :options="options"/>
      </li>
      <li>
        <span>异步tree：</span>
        <tree :data="asyncTree" :options="options" async :load="load"/>
      </li>
    </ul>
  </div>
</template>

<script>
import tree from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    tree
  },
  data () {
    return {
      options: {
        id: 'id',
        children: 'children',
        name: 'label'
      },
      asyncTree: [
        {
          label: 'test'
        },
        {
          label: 'test'
        },
        {
          label: 'test'
        },
        {
          label: 'test'
        }
      ],
      data: [
        {
          label: 'test'
        },
        {
          label: '一级 1',
          children: [
            {
              label: '二级 1-1',
              children: [
                {
                  label: '三级 1-1-1'
                }
              ]
            }
          ]
        },
        {
          label: '一级 2',
          children: [
            {
              label: '二级 2-1',
              children: [
                {
                  label: '三级 2-1-1'
                }
              ]
            },
            {
              label: '二级 2-2',
              children: [
                {
                  label: '三级 2-2-1'
                }
              ]
            }
          ]
        },
        {
          label: '一级 3',
          children: [
            {
              label: '二级 3-1',
              children: [
                {
                  label: '三级 3-1-1'
                }
              ]
            },
            {
              label: '二级 3-2',
              children: [
                {
                  label: '三级 3-2-1'
                }
              ]
            }
          ]
        }
      ]
    }
  },
  methods: {
    load (node, resolve) {
      console.log(node, resolve)
      setTimeout(() => {
        resolve([
          {
            label: '二级1'
          },
          {
            label: '二级2'
          },
          {
            label: '二级3'
          }
        ], { dsf: 123 })
      }, 1000)
    }
  }
}
</script>

<style lang="scss" scoped>
#app {
  width: 600px;
  // text-align: center;
  margin: 0 auto;
  ul {
    // display: flex;
    li {
      // width: 500px;
      // flex: 0 0 200px;
      border: 2px solid red;
      margin-bottom: 20px;
    }
  }
}
</style>
