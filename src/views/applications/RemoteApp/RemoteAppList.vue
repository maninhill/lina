<template>
  <GenericListPage :table-config="tableConfig" :header-actions="headerActions" />
</template>

<script>
import { GenericListPage } from '@/layout/components'
import { REMOTE_APP_TYPE_META_MAP, ALL_TYPES } from './const'

export default {
  components: {
    GenericListPage
  },
  data() {
    return {
      tableConfig: {
        url: '/api/v1/applications/remote-apps/',
        columns: [
          'name', 'type', 'asset', 'comment', 'actions'
        ],
        columnsMeta: {
          type: {
            displayKey: 'get_type_display'
          },
          asset: {
            formatter: function(row, column, cellValue, index) {
              return <a class='detail el-link el-link--success is-underline' href={ `/assets/assets/${cellValue}` }>{ row.asset_info.hostname }</a>
            }
          }
        },
        detailRoute: 'RemoteAppDetail',
        actions: {
          updateRoute: 'RemoteAppUpdate'
        }
      },
      headerActions: {
        hasCreate: false,
        hasBulkDelete: false,
        // createRoute: 'RemoteAppCreate',
        moreActionsTitle: '创建',
        extraMoreActions: this.genExtraMoreActions()
      }
    }
  },
  methods: {
    onCallback(type) {
      this.$router.push({ name: 'RemoteAppCreate', query: { type: type }})
    },
    genExtraMoreActions() {
      const extra_more_actions = []
      for (const value of ALL_TYPES) {
        const item = { ...REMOTE_APP_TYPE_META_MAP[value] }
        item.type = 'primary'
        item.can = true
        item.callback = this.onCallback.bind(this, value)
        extra_more_actions.push(item)
      }
      return extra_more_actions
    }
  }
}
</script>

<style lang="less" scoped>

</style>
