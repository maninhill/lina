<template>
  <GenericCreateUpdatePage :fields="fields" :initial="initial" :fields-meta="fieldsMeta" :url="url" />
</template>

<script>
import { AssetSelect } from '@/components'
import GenericCreateUpdatePage from '@/layout/components/GenericCreateUpdatePage'
export default {
  name: 'LabelCreateUpdate',
  components: {
    GenericCreateUpdatePage
  },
  data() {
    return {
      initial: {

      },
      fields: [
        ['', ['name', 'value', 'assets']]
      ],
      fieldsMeta: {
        assets: {
          type: 'assetSelect',
          component: AssetSelect,
          label: '资产',
          rules: [{
            required: false
          }]
        }
      },
      url: '/api/v1/assets/labels/'
    }
  }
}
</script>

<style>

</style>
