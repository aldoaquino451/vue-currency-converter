<script>


export default {
  name: "Curreny",
  
  props: {
    currencyResult: Object
  },

  computed: {
    currencyAmount() {
      return this.currencyResult.amount.toFixed(2); 
    },
    currencyAmountFinal() {
      return this.currencyResult.amountFinal.toFixed(2); 
    }
  },

}

</script>


<template>
  <div class="mb-4">
    <p class="m-0 small">{{ this.currencyResult.amount }} {{currencyResult.base}} è uguale a:</p>
    <p class="m-0 large">{{ this.currencyResult.amountFinal }} {{ currencyResult.baseFinal }}</p>
  </div>
</template>


<style lang="scss" scoped>

.small {
  font-size: 1em;
}
.large {
  font-size: 1.6em;
}
</style>