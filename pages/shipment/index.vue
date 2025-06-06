<template>
  <shipments-all />
</template>

<script>
import shipmentsAll from '@/components/shipments/shipmentsAll.vue'
export default {
  components: {
    shipmentsAll
  },
  layout: 'default.vue'
  // middleware: 'auth'
}
</script>
