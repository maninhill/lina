<template>
  <DataZTree ref="dataztree" :setting="treeSetting">
    <slot slot="rMenu">
      <li id="m_create" class="rmenu" tabindex="-1" @click="addTreeNode"><a><i class="fa fa-plus-square-o" /> {{ this.$t('tree.AddNode') }} </a></li>
      <li id="m_edit" class="rmenu" tabindex="-1" @click="editTreeNode"><a><i class="fa fa-pencil-square-o" /> {{ this.$t('tree.RenameNode') }} </a></li>
      <li id="m_del" class="rmenu" tabindex="-1" @click="removeTreeNode"><a><i class="fa fa-minus-square" /> {{ this.$t('tree.DeleteNode') }} </a></li>
      <slot name="rMenu" />
    </slot>
  </DataZTree>
</template>

<script>
import DataZTree from '../DataZTree'
import $ from '@/utils/jquery-vendor'
export default {
  name: 'AutoDataZTree',
  components: {
    DataZTree
  },
  props: {
    setting: {
      type: Object,
      default: () => {}
    }
  },
  data() {
    return {
      defaultSetting: {
        async: {
          enable: true,
          url: `${process.env.VUE_APP_BASE_API}${this.setting.treeUrl}`,
          autoParam: ['id=key', 'name=n', 'level=lv'],
          type: 'get'
        },
        callback: {
          onRightClick: this.onRightClick.bind(this),
          onRename: this.onRename.bind(this),
          onSelected: this.onSelected.bind(this),
          beforeDrop: this.beforeDrop.bind(this),
          onDrop: this.onDrop.bind(this),
          refresh: this.refresh.bind(this)
          // 尚未定义的函数
          // beforeClick
          // beforeDrag
          // onDrag
          // beforeAsync: this.defaultCallback.bind(this, 'beforeAsync')
        }
      },
      currentNode: '',
      currentNodeId: ''
    }
  },
  computed: {
    treeSetting() {
      return _.merge(this.defaultSetting, this.setting)
    },
    zTree() {
      return this.$refs.dataztree.zTree
    },
    rMenu() {
      return this.$refs.dataztree.rMenu
    }
  },
  methods: {
    setUrlParam: function(url, name, value) {
      var urlArray = url.split('?')
      if (urlArray.length === 1) {
        url += '?' + name + '=' + value
      } else {
        var oriParam = urlArray[1].split('&')
        var oriParamMap = {}
        $.each(oriParam, function(index, value) {
          var v = value.split('=')
          oriParamMap[v[0]] = v[1]
        })
        oriParamMap[name] = value
        url = urlArray[0] + '?'
        var newParam = []
        $.each(oriParamMap, function(index, value) {
          newParam.push(index + '=' + value)
        })
        url += newParam.join('&')
      }
      return url
    },
    editTreeNode: function() {
      this.hideRMenu()
      var currentNode = this.zTree.getSelectedNodes()[0]
      if (!currentNode) {
        return
      }
      if (currentNode) {
        currentNode.name = currentNode.meta.node.value
      }
      this.zTree.editName(currentNode)
    },
    hideRMenu: function() {
      if (this.rMenu) this.rMenu.css({ 'visibility': 'hidden' })
      $('body').unbind('mousedown', this.onBodyMouseDown)
    },
    // Request URL: http://localhost/api/v1/assets/assets/?node_id=d8212328-538d-41a6-bcfd-1e8cc7e3aed4&show_current_asset=null&draw=2&limit=15&offset=0&_=1587022917769

    onSelected: function(event, treeNode) {
      if (treeNode.meta.type === 'node') {
        this.currentNode = treeNode
        this.currentNodeId = treeNode.meta.node.id
        this.$emit('urlChange', `${this.setting.url}?node_id=${treeNode.meta.node.id}&show_current_asset=null`)
      } else if (treeNode.meta.type === 'asset') {
        this.$emit('urlChange', `${this.setting.url}?asset_id=${treeNode.meta.asset.id}&show_current_asset=null`)
      }
    },
    removeTreeNode: function() {
      this.hideRMenu()
      var currentNode = this.zTree.getSelectedNodes()[0]
      if (!currentNode) {
        return
      }
      this.$axios.delete(
        `${this.treeSetting.nodeUrl}${currentNode.meta.node.id}/`,
      ).then(
        this.zTree.removeNode(currentNode)
      )
    },
    onRename: function(event, treeId, treeNode, isCancel) {
      var url = `${this.treeSetting.nodeUrl}${this.currentNodeId}/`
      if (isCancel) {
        return
      }
      this.$axios.patch(
        url,
        { 'value': treeNode.name }
      ).then(res => {
        var assets_amount = treeNode.meta.node.assets_amount
        if (!assets_amount) {
          assets_amount = 0
        }
        treeNode.name = treeNode.name + ' (' + assets_amount + ')'
        this.zTree.updateNode(treeNode)
      })
    },
    onBodyMouseDown: function(event) {
      if (!(event.target.id === 'rMenu' || $(event.target).parents('#rMenu').length > 0)) {
        this.rMenu.css({ 'visibility': 'hidden' })
      }
    },
    showRMenu: function(type, x, y) {
      var offset = $('#ztree').offset()
      var scrollTop = document.querySelector('.treebox').scrollTop
      x -= offset.left
      y -= offset.top + scrollTop
      x += document.body.scrollLeft
      y += document.body.scrollTop + document.documentElement.scrollTop
      this.rMenu.css({ 'top': y + 'px', 'left': x + 'px', 'visibility': 'visible' })
      $('#rMenu ul').show()
      $('body').bind('mousedown', this.onBodyMouseDown)
    },
    onRightClick: function(event, treeId, treeNode) {
      if (!this.setting.showMenu) {
        return
      }
      if (!treeNode && event.target.tagName.toLowerCase() !== 'button' && $(event.target).parents('a').length === 0) {
        this.zTree.cancelSelectedNode()
        this.showRMenu('root', event.clientX, event.clientY)
      } else if (treeNode && !treeNode.noR) {
        this.zTree.selectNode(treeNode)
        this.showRMenu('node', event.clientX, event.clientY)
      }
    },
    beforeDrop: function(treeId, treeNodes, targetNode, moveType) {
      var treeNodesNames = []
      $.each(treeNodes, function(index, value) {
        treeNodesNames.push(value.name)
      })

      // TODO 修改默认确认框
      var msg = '你想移动节点: `' + treeNodesNames.join(',') + '` 到 `' + targetNode.name + '` 下吗?'
      return confirm(msg)
    },
    onDrop: function(event, treeId, treeNodes, targetNode, moveType) {
      var treeNodesIds = []
      $.each(treeNodes, function(index, value) {
        console.log(value)
        treeNodesIds.push(value.meta.node.id)
      })
      var the_url = `${this.treeSetting.nodeUrl}${targetNode.meta.node.id}/children/add/`
      this.$axios.put(
        the_url, {
          nodes: treeNodesIds
        }
      ).then((res) => {
        console.log(res)
      })
    },
    addTreeNode: function() {
      this.hideRMenu()
      var parentNode = this.zTree.getSelectedNodes()[0]
      if (!parentNode) {
        return
      }
      // http://localhost/api/v1/assets/nodes/85aa4ee2-0bd9-41db-9079-aa3646448d0c/children/
      var url = `${this.treeSetting.nodeUrl}${parentNode.meta.node.id}/children/`
      this.$axios.post(
        url, {}
      ).then(data => {
        var newNode = {
          id: data['key'],
          name: data['value'],
          pId: parentNode.id,
          meta: {
            'node': data
          }
        }
        newNode.checked = this.zTree.getSelectedNodes()[0].checked
        this.zTree.addNodes(parentNode, 0, newNode)
        var node = this.zTree.getNodeByParam('id', newNode.id, parentNode)
        this.zTree.editName(node)
      })
    },
    refresh: function() {
      this.$axios.post(
        '/api/v1/assets/nodes/00000000-0000-0000-0000-000000000000/tasks/',
        { action: 'refresh_cache' }
      )
    }
  }
}
</script>

<style lang='less' scoped>
  .rmenu  > a {
    border-radius: 3px;
    color: inherit;
    line-height: 25px;
    margin: 4px;
    text-align: left;
    font-weight: normal;
    display: block;
    padding: 3px 20px;
    clear: both;
    white-space: nowrap;
  }
  .rmenu>a:hover, .dropdown-menu>a:focus {
    color: #262626;
    text-decoration: none;
    background-color: #f5f5f5;
  }
</style>
