<template>
  <GenericDetailPage :actions="actions" :object.sync="TaskDetail" :active-menu.sync="config.activeMenu" v-bind="config" v-on="$listeners">
    <keep-alive>
      <component :is="config.activeMenu" :object="TaskDetail" />
    </keep-alive>
  </GenericDetailPage>
</template>

<script>
import { GenericDetailPage, TabPage } from '@/layout/components'
import Detail from './Detail.vue'
export default {
  components: {
    GenericDetailPage,
    TabPage,
    Detail
  },
  data() {
    return {
      TaskDetail: {},
      actions: {
        detailApiUrl: 'api/v1/assets/platform/details/1/'
      },
      config: {
        title: this.$t('assets.commandFilterDetail'),
        activeMenu: 'Detail',
        submenu: [
          {
            title: this.$t('assets.detail'),
            name: 'Detail'
          }
        ],
        hasRightSide: false
      }
    }
  }
}
</script>

<style scoped>

</style>
