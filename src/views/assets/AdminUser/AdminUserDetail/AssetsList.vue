<template>
  <el-row :gutter="20">
    <el-col :span="14">
      <ListTable ref="ListTable" :table-config="tableConfig" :header-actions="headerActions" />
    </el-col>
    <el-col :span="10">
      <QuickActions type="primary" :actions="quickActions" />
    </el-col>
  </el-row>
</template>

<script>
import ListTable from '@/components/ListTable/index'
import { CustomActionsFormatter, DateFormatter } from '@/components/ListTable/formatters'
import QuickActions from '@/components/QuickActions/index'

export default {
  name: 'Detail',
  components: {
    QuickActions,
    ListTable
  },
  props: {
    object: {
      type: Object,
      default: () => {}
    }
  },
  data() {
    return {
      quickActions: [
        {
          title: this.$t('assets.TestAssetsConnective'),
          attrs: {
            type: 'primary',
            label: this.$tc('Test')
          },
          callbacks: {
            click: function() {
              this.$axios.post(
                `api/v1/assets/admin-users/${this.object.id}/tasks/`,
                { action: 'test' }
              ).then(res => {
                window.open(`/ops/celery/task/${res.task}/log/`, '', 'width=900,height=600')
              }
              )
            }.bind(this)
          }
        }
      ],
      tableConfig: {
        url: `/api/v1/assets/asset-users/?prefer_id=${this.object.id}&prefer=admin_user&latest=1`,
        columns: [
          {
            prop: 'hostname',
            label: this.$t('assets.hostname')
          },
          {
            prop: 'ip',
            label: this.$t('assets.ip')
          },
          {
            prop: 'username',
            label: this.$t('assets.username')
          },
          {
            prop: 'version',
            label: this.$t('assets.version')
          },
          {
            prop: 'date_created',
            label: this.$t('assets.date_created'),
            formatter: DateFormatter
          },
          {
            prop: 'id',
            label: this.$tc('Action'),
            align: 'center',
            formatter: CustomActionsFormatter,
            actions: {
              extraActions: [
                {
                  name: this.$tc('delete'),
                  title: this.$tc('delete'),
                  callback: (val) => {
                    this.$axios.delete(`/api/v1/assets/asset-users/${val.cellValue}/`).then(
                      this.$refs.ListTable.reloadTable()
                    )
                  }
                },
                {
                  name: this.$tc('Test'),
                  title: this.$tc('Test'),
                  callback: (val) => {
                    console.log('Test')
                  }
                }
              ]
            }
          }
        ]
      },
      headerActions: {
        hasRightActions: false,
        hasExport: false,
        hasImport: false,
        hasRefresh: false,
        hasBulkDelete: false,
        hasSearch: true,
        hasCreate: false
      }
    }
  },
  computed: {

  },
  mounted() {
  },
  methods: {
  }
}
</script>

<style lang='less' scoped>

</style>
