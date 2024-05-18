<script>
import axios from "axios";
import Search from "./components/Search.vue";
import Result from "./components/Result.vue";
import Test from "./components/Test.vue";

export default {
  components: {
    Search,
    Result,
    Test,
  },

  data() {
    return {
      baseUrl: "https://api.frankfurter.app/",
      currencies: {},
      data: {
        up: { amount: null, currency: "EUR" },
        down: { amount: null, currency: "USD" },
        result: { small: "up", large: "down" },
      },
      // dataChart: { from: "", to: "", dates: [], values: [] },
      // windowWidth: window.innerWidth,
    };
  },

  methods: {
    // parte una chiamata api diversa in base al tpo di input e alla componente
    search(amount, currency, position, isSelect) {
      if (position == "up") {
        if (isSelect) {
          this.getConvertion(amount, currency, this.data.down.currency, "up");
          // this.getDataChart(currency, this.data.down.currency);
        }

        if (!isSelect) {
          this.getConvertion(amount, currency, this.data["down"].currency, position);
          // this.getDataChart(currency, this.data["down"].currency);
        }
      }

      if (position == "down") {
        if (isSelect) {
          this.getConvertion(this.data.up.amount, this.data.up.currency, currency, "up");
          // this.getDataChart(currency, this.data.up.currency);
        }

        if (!isSelect) {
          this.getConvertion(amount, currency, this.data["up"].currency, position);
          // this.getDataChart(currency, this.data["up"].currency);
        }
      }
    },

    // ottengo un oggetto con i dati che verrano usati per renderizzare il grafico
    getDataChart(from, to) {
      const now = new Date();

      if (this.windowWidth >= 768) now.setMonth(now.getMonth() - 1);
      else now.setDate(now.getDate() - 7);

      let date = now.toLocaleDateString("us-US").split("/").reverse();
      date = date.map((el) => el.padStart(2, "0")).join("-");
      date = date + "..";

      axios
        .get(this.baseUrl + date, { params: { amount: 1, from: from, to: to } })
        .then((res) => {
          this.dataChart.from = from;
          this.dataChart.to = to;

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
        });
    },

    // ottengo un oggetto con tutti i dati da stampare dentro Result e Search
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

    // ottengo la lista di monete da stampare nella select
    getCurrencies() {
      axios.get(this.baseUrl + "currencies").then((res) => {
        this.currencies = res.data;
      });
    },
  },

  watch: {
    windowWidth(newValue, oldValue) {
      if (oldValue < 768 && newValue >= 768) {
        this.getDataChart("EUR", "USD");
      } else if (oldValue >= 768 && newValue < 768) {
        this.getDataChart("EUR", "USD");
      }
    },
  },

  mounted() {
    // window.addEventListener("resize", () => (this.windowWidth = window.innerWidth));
    this.getCurrencies();
    this.getConvertion(1, "EUR", "USD", "up");
    // this.getDataChart("EUR", "USD");
  },

  beforeDestroy() {
    window.removeEventListener("resize", this.handleResize);
  },
};
</script>

<template>
  <div class="my-container">
    <h1 class="text-center mb-3 mb-md-2">Currency Converter</h1>
    <div>
      <Result :data="this.data" />
    </div>
    <div class="d-flex flex-column gap-4 gap-md-3">
      <Search
        v-for="n in ['up', 'down']"
        :position="n"
        :data="this.data"
        :currencies="this.currencies"
        @search="search"
      />
    </div>
    <!-- <Test
      :dataChart="this.dataChart"
      :chartFromTo="this.dataChart.from + '|' + this.dataChart.to"
    /> -->
  </div>
</template>

<style lang="scss">
.my-container {
  max-width: 1000px;
  width: 90%;
  margin: 0 auto;
  min-height: 500px;
  border-radius: 40px;
  padding: 50px;
  background-color: #252525;
}

@media screen and (max-width: 767px) {
  .my-container {
    width: 100%;
    min-height: 500px;
    border: 0;
    border-radius: 0;
    padding: 20px 20px 50px;
  }
}
</style>
