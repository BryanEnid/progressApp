<template>
<div>
  <component v-bind:is="component"></component>
  
  
</div>
  
</template>

<script>
import ExpensesCalculator from './components/ExpensesCalculator.vue'
import LoginApp from './components/Login.vue'

  export default {
    components: {
     'expenses': ExpensesCalculator,
     'login': LoginApp
    },
    data(){
      return {
        component: 'expenses'
      }
    }
  }
</script>

<style>


</style>
