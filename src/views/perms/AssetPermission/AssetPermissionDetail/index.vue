<template>
  <GenericDetailPage :object.sync="AssetPermission" :active-menu.sync="config.activeMenu" v-bind="config" v-on="$listeners" @tab-click="TabClick">
    <component :is="config.activeMenu" :object="AssetPermission" />
  </GenericDetailPage>
</template>

<script>
import { GenericDetailPage, TabPage } from '@/layout/components'
import AssetPermissionDetail from './AssetPermissionDetail'
import AssetPermissionUser from './AssetPermissionUser'
import AssetPermissionAsset from './AssetPermissionAsset'

export default {
  components: {
    GenericDetailPage,
    AssetPermissionDetail,
    AssetPermissionUser,
    AssetPermissionAsset,
    TabPage
  },
  data() {
    return {
      AssetPermission: {
        name: '', users_amount: 0, user_groups_amount: 0, assets_amount: 0, nodes_amount: 0, system_users_amount: 0,
        date_start: '', date_expired: ''
      },
      config: {
        activeMenu: 'AssetPermissionDetail',
        submenu: [
          {
            title: this.$t('perms.AssetPermissionDetail'),
            name: 'AssetPermissionDetail'
          },
          {
            title: this.$t('perms.UsersAndUserGroups'),
            name: 'AssetPermissionUser'
          },
          {
            title: this.$t('perms.AssetAndNode'),
            name: 'AssetPermissionAsset'
          }
        ]
      }
    }
  },
  methods: {
    TabClick(tab) {
      if (tab.name !== 'AssetPermissionDetail') {
        this.$set(this.config, 'hasRightSide', false)
      } else {
        this.$set(this.config, 'hasRightSide', true)
      }
    }
  }
}
</script>

<style scoped>

</style>
