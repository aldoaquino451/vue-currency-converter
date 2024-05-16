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
        colors: ["#fff"],

        x: {
          show: true,
          format: "dd MMM",
          formatter: undefined,
        },
        theme: {
          mode: "dark",
          palette: "palette9",
          monochrome: {
            enabled: true,
            color: "#ffffff",
            shadeTo: "light",
            shadeIntensity: 0.8,
          },
        },
        chart: {
          height: "auto",
          type: "area",
          fontFamily: "Space Grotesk, Arial, sans-serif",
          redrawOnParentResize: true,
          offsetX: -5,
          offsetY: 10,
          width: "95%",
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
            color: "#d4d4d4",
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
      if (this.dataChart.from === from) return;

      // let lastMonth = new Date();
      // lastMonth.setMonth(lastMonth.getMonth() - 1);
      // lastMonth = lastMonth.toLocaleDateString("us-US");
      // lastMonth = lastMonth.split("/").reverse();
      // lastMonth = lastMonth.map((el) => el.padStart(2, "0")).join("-");
      // const date = lastMonth + "..";

      let lastWeek = new Date();
      lastWeek.setDate(lastWeek.getDate() - 7);
      lastWeek = lastWeek.toLocaleDateString("us-US");
      lastWeek = lastWeek.split("/").reverse();
      lastWeek = lastWeek.map((el) => el.padStart(2, "0")).join("-");
      const date = lastWeek + "..";

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
          console.log(this.data);
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

    <div id="chart"></div>
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

#chart {
  margin-top: 50px;
  color: #d4d4d4;
  display: flex;
  justify-content: center;
  background-color: #424242;
  border-radius: 40px;
  padding: 20px 0;
}

@media screen and (max-width: 767px) {
  .my-container {
    width: 100%;
    min-height: 500px;
    border: 0;
    border-radius: 0;
    padding: 40px 20px;
  }

  #chart {
    border-radius: 10px;
    padding: 15px 0;
  }
}
</style>
