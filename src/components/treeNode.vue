<template>
  <li class="treeNode">
    <div class="icon" @click="collapse">
      <i
        :class="['el-icon-caret-right',  'el-icon', node.collapse ? 'rotate' : '']"
        v-show="node.hasChildren">
      </i>
      <i
        :class="['el-icon-loading']"
        v-show="node.loading">
      </i>
    </div>
    <el-checkbox
      v-model="node.checked"
      :disabled="node.disabled"
      :label="node.id"
      :indeterminate="node.indeterminate"
      @change="change">
      <span class="name">{{ `${node.name} --- loading: ${node.loading}`}}</span>
    </el-checkbox>
    <el-collapse-transition>
      <div class="node-children" v-if="node.collapse">
        <slot></slot>
      </div>
    </el-collapse-transition>
  </li>
</template>

<script>
export default {
  name: 'treeNode',
  components: {},
  props: {
    data: {
      type: Object,
      default () {
        return {}
      }
    },
    treeOptions: {
      type: Object,
      default () {
        return {
          id: 'id',
          name: 'name',
          children: 'children'
        }
      }
    }
  },
  inject: {
    tree: {}
  },
  watch: {
    data: {
      handler (val) {
      },
      deep: true,
      immediate: true
    }
  },
  data () {
    return {
      node: this.data.treeNodeOptions
    }
  },
  created () {
    console.log(this.data)
  },
  methods: {
    collapse () {
      if (this.tree.accordion) {
        const { children } = this.treeOptions
        const nodeList = this.node.parent ? this.node.parent[children] : this.tree.treeData
        const find = nodeList.filter(item => item.treeNodeOptions.collapse)

        if (find.length) {
          find.map(item => {
            item.treeNodeOptions.collapse = false
          })
          this.node.collapse = !this.node.collapse
        } else {
          this.node.collapse = !this.node.collapse
        }
      } else {
        this.node.collapse = !this.node.collapse
      }

      if (this.node.collapse) {
        if (this.tree.async && this.tree.load) {
          this.tree.asyncTree(this.data)
        }
      }

      this.$emit('collapse', this.node)
    },
    change (val, $event) {
      console.log(val, $event, this.data)
      this.tree.setNodeChecked(this.data, val)
    }
  }
}
</script>

<style lang="scss" scoped>
.treeNode {
  margin-bottom: 3px;
  .icon {
    margin-right: 3px;
    display: inline-block;
    .el-icon-caret-right {
      width: 16px;
      display: inline-block;
    }
    .el-icon {
      font-size: 14px;
      color: #c0c4cc;
      transition: 0.15s linear;
    }
    .rotate {
      transform: rotate(90deg);
    }
    // .node-children {
    //   // transition: 0.1s linear;
    // }
    // border: 2px solid red;
  }
}
</style>
