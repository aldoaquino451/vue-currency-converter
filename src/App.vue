<script>
import axios from "axios";
import Search from "./components/Search.vue";
import Result from "./components/Result.vue";
import Chart from "./components/Chart.vue";

export default {
  components: {
    Search,
    Result,
    Chart,
  },
  data() {
    return {
      baseUrl: "https://api.frankfurter.app/",
      currencies: {},
      data: {
        up: { amount: null, currency: "" },
        down: { amount: null, currency: "" },
        result: { small: "up", large: "down" },
      },
      dataChart: { from: "", to: "", dates: [], values: [] },
    };
  },
  methods: {
    getDataChart(from, to) {
      let lastMonth = new Date();
      lastMonth.setMonth(lastMonth.getMonth() - 1);
      lastMonth = lastMonth.toLocaleDateString("us-US");
      lastMonth = lastMonth.split("/").reverse();
      lastMonth = lastMonth.map((el) => el.padStart(2, "0")).join("-");
      const date = lastMonth + "..";

      axios.get(this.baseUrl + date, { params: { from: from, to: to } }).then((res) => {
        this.dataChart["from"] = from;
        this.dataChart["to"] = to;

        this.dataChart.dates = [];
        this.dataChart.values = [];

        Object.entries(res.data.rates).forEach((el) => {
          const newDate = new Date(el[0]);
          const dateFormatted = newDate.toLocaleDateString("it-IT", {
            month: "short",
            day: "numeric",
          });

          this.dataChart.dates.push(dateFormatted);
          this.dataChart.values.push(el[1][to]);
        });
        console.log(this.dataChart);
      });
    },

    getCurrencies() {
      axios.get("https://api.frankfurter.app/currencies").then((res) => {
        this.currencies = res.data;
      });
    },

    getConvertion(amount, from, to, position) {
      axios
        .get(this.baseUrl + "latest", { params: { amount: amount, from: from, to: to } })
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
      if (position === "up") {
        this.getConvertion(amount, currency, this.data.down.currency, position);
        this.getDataChart(currency, this.data.down.currency);
      } else {
        this.getConvertion(amount, currency, this.data.up.currency, position);
        this.getDataChart(currency, this.data.up.currency);
      }
    },
  },
  created() {
    this.getCurrencies();
    this.getConvertion(1, "EUR", "USD", "up");
    this.getDataChart("EUR", "USD");
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

    <div style="min-height: 300px">
      <Chart :data="this.dataChart" />
    </div>
  </div>
</template>

<style lang="scss">
.my-container {
  max-width: 1000px;
  width: 90%;
  margin: 150px auto;
  min-height: 500px;
  border: 1px solid black;
  border-radius: 40px;
  padding: 50px;
}
</style>
