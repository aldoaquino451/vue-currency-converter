<script>
import axios from "axios";
import Search from "./components/Search.vue";
import Currency from "./components/Currency.vue";

export default {
  components: {
    Search,
    Currency,
  },

  data() {
    return {
      baseUrl: "https://api.frankfurter.app/latest",
      amount: 1,
      from: 'EUR',
      to: 'USD',
      currencyResult: {}, 
      currencies: {},
    }
  },

  methods: {
    getCurrencies() {
      axios.get('https://api.frankfurter.app/currencies')
        .then( res => {
          this.currencies = res.data;
        })
    },

    getConvertion(amount,from,to) {
      axios.get(this.baseUrl, { params: {
        amount: amount,
        from: from,
        to: to
      }}) 
        .then( res => {
          this.currencyResult = res.data;

          const rates = Object.entries(this.currencyResult.rates);

          // rates.forEach((currency, key) => {
          //   console.log(currency[0], currency[1]);
          // });
          
          for (const [key, value] of Object.entries(this.currencyResult.rates)) {
            this.currencyResult['baseFinal'] = key;
            this.currencyResult['amountFinal'] = value;
          }
        })
    }  

  },

  mounted() {
    this.getCurrencies(); 
    this.getConvertion(1, 'EUR', 'USD');
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
      <Search :currencies="this.currencies" />
      <Search :currencies="this.currencies" />
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
