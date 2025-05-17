<template>
  <departments-all />
</template>

<script>
import departmentsAll from '@/components/departments/departmentsAll.vue'
export default {
  components: {
    departmentsAll
  },
  layout: 'default.vue'
  // middleware: 'auth'
}
</script>
