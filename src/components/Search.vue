<script>

export default {
  name: "Search",

  props: {
    currencies : Object 
  },

  data() {
    return {
      currencyToSearch: '',
      currencyKey: '$'
    }
  },

}
</script>


<template>

  <div class="d-flex gap-3">
    <div class="input-group">
      <input type="number" class="form-control" step=0.00001 v-model="currencyToSearch">
      <span class="input-group-text">{{ currencyKey }}</span>
    </div>

    <select class="form-select">
      <option selected>...</option>
      <option :value="key" v-for="(currency, key) in currencies">{{ currency }}</option>
    </select>
  </div>
    
</template>


<style lang="scss" scoped>

</style>