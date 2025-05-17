<template>
  <employees-all />
</template>

<script>
import employeesAll from '@/components/employee/employeesAll.vue'
export default {
  components: {
    employeesAll
  },
  layout: 'default.vue'
  // middleware: 'auth'
}
</script>
