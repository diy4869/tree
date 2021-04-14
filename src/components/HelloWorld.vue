<script>
import ElCollapseTransition from 'element-ui/src/transitions/collapse-transition'
// import { treeOptions } from './options'
import treeNode from './treeNode'
import cloneDeep from 'lodash/cloneDeep'

export default {
  name: 'tree',
  components: {
    ElCollapseTransition,
    treeNode
  },
  props: {
    // 是否同级只展开一个
    accordion: {
      type: Boolean,
      default: false
    },
    // 是否异步tree
    async: {
      type: Boolean
    },
    load: {
      type: Function,
      default () {
        return function () {}
      }
    },
    // 是否显示checkbox
    checkbox: {
      type: Boolean,
      default: false
    },
    data: {
      type: Array,
      default () {
        return []
      }
    },
    options: {
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
  provide () {
    return {
      tree: this
    }
  },
  data () {
    return {
      treeData: cloneDeep(this.data),
      nodeCheckedList: []
    }
  },
  created () {
    this.treeData = this.transform()
  },
  render (h) {
    const renderTree = (treeData = this.treeData) => {
      const data = treeData.map((current) => {
        if (Array.isArray(current?.children)) {
          return h('tree-node', {
            props: {
              hasChildren: true,
              data: current
            }
          }, [
            h('ul', {
              style: {
                paddingLeft: '20px'
              },
              on: {
                click (e) {
                  console.log(e)
                }
              }
            }, renderTree(current?.children))
          ])
        } else {
          return h('tree-node', {
            props: {
              data: current
            }
          })
        }
      })

      return data
    }

    return h('ul', {
      on: {
        click (e) {
          console.log(e)
        }
      }
    }, renderTree())
  },
  methods: {
    transform (data = this.data) {
      let nodeId = 1
      let depth = 1

      const { children, id, name } = this.options
      const renderTree = (data, parent = undefined) => {
        const result = data.reduce((total, current) => {
          depth = parent === undefined ? 1 : ++depth
          const hasChildren = !!current[children]

          current.treeNodeOptions = {
            disabled: false,
            checked: false,
            collapse: false,
            loading: false,
            loaded: false,
            indeterminate: false,
            childrenChecked: [],
            id: current?.[id] ?? nodeId++,
            name: current[name],
            parent,
            depth,
            hasChildren: this.async && typeof this.load === 'function' ? true : hasChildren
          }

          // eslint-disable-next-line no-unused-expressions
          Array.isArray(current?.[children]) ? renderTree(current[children], current) : current

          return total.concat(current)
        }, [])

        depth = 1
        return result
      }

      return renderTree(data)
    },
    setData (node, childrenData) {
      const { children } = this.options

      const deep = (data = this.treeData) => {
        return data.map(item => {
          if (node.treeNodeOptions.id === item.treeNodeOptions.id) {
            item.children = childrenData
          } else {
            deep(item[children])
          }

          return item
        })
      }
      deep()
    },
    asyncTree (node) {
      node.treeNodeOptions.loading = true

      return new Promise((resolve, reject) => {
        this.load(node, resolve, reject)
      }).then(data => {
        console.log(data)
        node.treeNodeOptions.loading = false
        this.setData(node, data)
      }).catch(err => {
        console.log(err)
        node.treeNodeOptions.loading = false
      })
    },
    setNodeChecked (node, checked) {
      const treeNodeOptions = node.treeNodeOptions

      const deep = (data = this.treeData, currentNode = treeNodeOptions) => {
        return data.map(item => {
          const node = item.treeNodeOptions
          if (node.id === currentNode.id) {
            node.checked = checked
            if (node.indeterminate) {
              node.indeterminate = false
            }
            if (item.children) {
              // item.treeNodeOptions.childrenChecked = item.children.map(item => item.treeNodeOptions.id)
              item.children.map(children => {
                return deep(item.children, children.treeNodeOptions)
              })
            }
          }

          return item
        })
      }

      /**
       * 如果存在父节点，则说明是中间节点或者最下级节点，需要向上递归，并给上级节点设置半选或选中状态
       * 否则则说明是跟节点，直接递归选中所有子节点
       */
      if (treeNodeOptions.parent) {
        const upDeep = (parentNode = treeNodeOptions.parent) => {
          parentNode.treeNodeOptions.checked = parentNode.children.every(item => {
            return item.treeNodeOptions.checked
          })

          parentNode.treeNodeOptions.indeterminate = parentNode.children.some(item => {
            return item.treeNodeOptions.checked
          })

          if (node.children) {
            node.children.map(children => {
              return deep(node.children, children.treeNodeOptions)
            })
          }

          // 如果checked和indeterminate同时成立，则以checked为主
          if (parentNode.treeNodeOptions.checked) {
            parentNode.treeNodeOptions.indeterminate = false
          }

          if (parentNode.treeNodeOptions.parent) {
            return upDeep(parentNode.treeNodeOptions.parent)
          } else {
            return parentNode
          }
        }
        upDeep()
      } else {
        deep()
      }
    },
    /**
     * @param { Array<string | number> } 设置的节点数组
     */
    setCheckedKeys (state, id) {
      console.log(state, id)

      const deep = (data = this.treeData) => {
        let result
        for (let i = 0; i < id.length; i++) {
          result = data.map(current => {
            const treeNodeOptions = current.treeNodeOptions
            if (id[i] === treeNodeOptions.id) {
              current.treeNodeOptions.checked = true
            }

            console.log(current)
            return current
          })
        }

        return result
      }

      this.treeData = deep()
      // console.log(data, this.treeData)
    }
  }
}
</script>

<style lang="scss" scoped>
ul {
  // border: 2px solid red;
  list-style: none;
}
</style>
