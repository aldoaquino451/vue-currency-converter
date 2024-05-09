<script>
import axios from "axios";
import Search from "./components/Search.vue";
import Result from "./components/Result.vue";
import ApexCharts from "apexcharts";

export default {
  components: {
    Search,
    Result,
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
      dataChart: { from: "", to: "", dates: [], values: [] },
      timeOut: null,
    };
  },

  methods: {
    getChart() {
      const chart = document.querySelector("#chart");
      chart.innerHTML = null;

      const options = {
        chart: {
          height: 280,
          type: "area",
        },
        dataLabels: {
          enabled: false,
        },
        series: [
          {
            name: "Series 1",
            data: this.dataChart.values,
          },
        ],
        fill: {
          type: "gradient",
          gradient: {
            shadeIntensity: 1,
            opacityFrom: 0.7,
            opacityTo: 0.9,
            stops: [0, 90, 100],
          },
        },
        title: {
          text: `Rapporto fra ${this.dataChart.from} e ${this.dataChart.to} nell'ultimo mese`,
          align: "center",
          margin: 60,
          offsetX: 0,
          offsetY: 0,
          floating: true,
          style: {
            fontSize: "14px",
            fontWeight: "bold",
            fontFamily: "Space Grotesk",
            color: "#263238",
          },
        },
        xaxis: {
          categories: this.dataChart.dates,
        },
      };

      const chartObj = new ApexCharts(chart, options);

      chartObj.render();
    },

    getDataChart(from, to) {
      let lastMonth = new Date();
      lastMonth.setMonth(lastMonth.getMonth() - 1);
      lastMonth = lastMonth.toLocaleDateString("us-US");
      lastMonth = lastMonth.split("/").reverse();
      lastMonth = lastMonth.map((el) => el.padStart(2, "0")).join("-");
      const date = lastMonth + "..";

      axios
        .get(this.baseUrl + date, { params: { amount: 1, from: from, to: to } })
        .then((res) => {
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

          this.getChart();
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
      clearTimeout(this.timeOut);

      this.timeOut = setTimeout(() => {
        if (position === "up") {
          this.getConvertion(amount, currency, this.data.down.currency, position);
          this.getDataChart(currency, this.data.down.currency);
        } else {
          this.getConvertion(amount, currency, this.data.up.currency, position);
          this.getDataChart(currency, this.data.up.currency);
        }
      }, 300);
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
      <div id="chart"></div>
    </div>
  </div>
</template>

<style lang="scss">
.my-container {
  max-width: 1000px;
  width: 90%;
  margin: 0 auto;
  min-height: 500px;
  border: 1px solid black;
  border-radius: 40px;
  padding: 50px;
}

#chart {
  margin-top: 50px;
}

@media screen and (max-width: 767px) {
  .my-container {
    width: 100%;
    min-height: 500px;
    border: 0;
    padding: 40px 20px;
  }
  #chart {
    margin-top: 50px;
    scale: 1.03;
    transform: translateX(-1%);
  }
}
</style>
