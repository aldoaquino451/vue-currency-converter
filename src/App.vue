<script>
import axios from "axios";
import { state } from './state.js';

import Search from "./components/Search.vue";
import Currency from "./components/Currency.vue";

export default {
  emit: ['getConvertion'],
  components: {
    Search,
    Currency,
  },

  data() {
    return {
      state,
      baseUrl: "https://api.frankfurter.app/latest",
      amount: 1,
      currencyResult: {},
      currencies: {},
    }
  },

  methods: {
    getCurrencies() {
      axios.get('https://api.frankfurter.app/currencies')
        .then(res => {
          this.currencies = res.data;
        })
        .catch(err => {
          console.error(err.message);
        })
    },

    getConvertion(amount, from, to, inputNumber) {

      console.log(amount, from, to);

      axios.get(this.baseUrl, {
        params: {
          amount: amount,
          from: from,
          to: to
        }
      })
        .then(res => {


          this.currencyResult = res.data;

          const rates = Object.entries(this.currencyResult.rates);

          for (const [key, value] of rates) {
            this.currencyResult['baseFinal'] = key;
            this.currencyResult['amountFinal'] = value;
          }

          if (inputNumber === 1) {
            this.state.currencyToSearch_1 = res.data.amount
            this.state.currencyToSearch_2 = res.data.amountFinal
          } else {
            this.state.currencyToSearch_2 = res.data.amount
            this.state.currencyToSearch_1 = res.data.amountFinal
          }


        })
        .catch(err => {
          console.error(err.message);
        })
    }

  },

  mounted() {
    this.getCurrencies();
    this.getConvertion(1, this.state.fromKey, this.state.toKey, 1);
  },
}


</script>

<template>

  <div class="my-container">

    <h1 class="text-center">Currency Converter</h1>

    <div>
      <Currency :currencyResult="this.currencyResult" />
    </div>

    <div class="d-flex flex-column gap-3">
      <Search :currencies="this.currencies" v-for="n in 2" :inputNumber="n" @getConvertion="getConvertion" />
    </div>

  </div>

</template>



<style lang="scss">
.my-container {
  max-width: 1000px;
  width: 70%;
  margin: 150px auto;
  min-height: 500px;
  border: 1px solid black;
  border-radius: 40px;
  padding: 50px;
}
</style>
