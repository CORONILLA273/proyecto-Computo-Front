<template>
  <products-all />
</template>

<script>
import productsAll from '@/components/products/productsAll.vue'
export default {
  components: {
    productsAll
  },
  layout: 'default.vue'
  // middleware: 'auth'
}
</script>
