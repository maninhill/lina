<template>
  <el-tabs>
    <el-tab-pane :label="this.$t('tickets.MyTickets')">
      <GenericListPage :table-config="tableConfig" :header-actions="headerActions" />
    </el-tab-pane>
    <el-tab-pane :label="this.$t('tickets.AssignedMe')" />

  </el-tabs>
</template>

<script>
import { GenericListPage } from '@/layout/components'
import { DetailFormatter } from '@/components/ListTable/formatters/index'

export default {
  components: {
    GenericListPage
  },
  data() {
    return {
      tableConfig: {
        url: '/api/v1/tickets/tickets/',
        axiosConfig: {
          raw: 1,
          params: {
            assign: 0,
            display: 1,
            draw: 1
          }
        },
        columns: [
          {
            prop: 'title',
            label: this.$t('tickets.title'),
            formatter: DetailFormatter,
            sortable: 'custom',
            route: 'UserDetail'
          },
          {
            prop: 'user_display',
            label: this.$t('tickets.user'),
            sortable: 'custom'
          },
          {
            prop: 'type_display',
            label: this.$t('tickets.type'),
            sortable: 'custom'
          },
          {
            prop: 'status',
            label: this.$t('tickets.status'),
            sortable: 'custom'
          },
          {
            prop: 'date_created',
            label: this.$t('tickets.date'),
            sortable: 'custom'
          }
        ]
      },
      headerActions: {
        hasCreate: false,
        hasBulkDelete: false
      }
    }
  }
}
</script>

<style lang="less" scoped>

</style>
