<template>
  <suppliers-all />
</template>

<script>
import suppliersAll from '@/components/suppliers/suppliersAll.vue'
export default {
  components: {
    suppliersAll
  },
  layout: 'default.vue'
  // middleware: 'auth'
}
</script>
