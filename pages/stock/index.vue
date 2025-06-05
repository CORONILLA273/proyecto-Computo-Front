<template>
  <stocks-all />
</template>

<script>
import stocksAll from '@/components/stocks/stocksAll.vue'
export default {
  components: {
    stocksAll
  },
  layout: 'default.vue'
  // middleware: 'auth'
}
</script>
