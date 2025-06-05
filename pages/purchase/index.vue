<template>
  <purchases-all />
</template>

<script>
import purchasesAll from '@/components/purchases/purchasesAll.vue'
export default {
  components: {
    purchasesAll
  },
  layout: 'default.vue'
  // middleware: 'auth'
}
</script>
