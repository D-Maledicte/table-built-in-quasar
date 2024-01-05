<template>
  <q-layout>
    <q-page-container>
      <Table />
    </q-page-container>
  </q-layout>
</template>

<script>
import Table from './components/TableComponent.vue'

export default {
  name: 'LayoutDefault',

  components: {
    Table
  },

  setup () {
  }
}
</script>
