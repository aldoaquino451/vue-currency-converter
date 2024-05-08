<script>
import axios from "axios";
import Search from "./components/Search.vue";
import Result from "./components/Result.vue";
export default {
  components: {
    Search,
    Result,
  },
  data() {
    return {
      baseUrl: "https://api.frankfurter.app/latest",
      currencies: {},
      data: {
        up: { amount: null, currency: "" },
        down: { amount: null, currency: "" },
        result: { small: "up", large: "down" },
      },
    };
  },
  methods: {
    getCurrencies() {
      axios.get("https://api.frankfurter.app/currencies").then((res) => {
        this.currencies = res.data;
      });
    },
    getConvertion(amount, from, to, position) {
      axios
        .get(this.baseUrl, { params: { amount: amount, from: from, to: to } })
        .then((res) => {
          const from = {};
          const to = {};

          from["amount"] = res.data.amount;
          from["currency"] = res.data.base;

          for (const [key, value] of Object.entries(res.data.rates)) {
            to["amount"] = value;
            to["currency"] = key;
          }

          this.data["up"] = position === "up" ? from : to;
          this.data["down"] = position === "up" ? to : from;

          this.data["result"] = {
            small: position,
            large: position === "up" ? "down" : "up",
          };
        });
    },
    search(amount, currency, position) {
      if (position === "up")
        this.getConvertion(amount, currency, this.data.down.currency, position);
      else this.getConvertion(amount, currency, this.data.up.currency, position);
    },
  },
  created() {
    this.getCurrencies();
    this.getConvertion(1, "EUR", "USD", "up");
  },
};
</script>

<template>
  <div class="my-container">
    <h1 class="text-center">Currency Converter</h1>
    <div>
      <Result :data="this.data" />
    </div>
    <div class="d-flex flex-column gap-3">
      <Search
        v-for="n in ['up', 'down']"
        :position="n"
        :data="this.data"
        :currencies="this.currencies"
        @search="search"
      />
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
