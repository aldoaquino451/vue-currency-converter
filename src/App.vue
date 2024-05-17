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
      dataChart: { from: null, to: null, dates: [], values: [] },
      timeOut: null,
      windowWidth: window.innerWidth,
    };
  },

  methods: {
    getChart(dataChart) {
      console.log("getChart");
      console.log(this.dataChart.values);
      console.log(this.dataChart.dates);

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
            data: dataChart.values,
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
          text:
            "Rapporto fra " + dataChart.from + " e " + dataChart.to + " nell'ultimo mese",
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
          categories: dataChart.dates,
        },
      };

      const chartObj = new ApexCharts(chart, options);

      chartObj.render();
    },

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

          this.getChart(this.dataChart);
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

    search(amount, currency, position, isSelect) {
      if (position === "up" && isSelect) {
        this.getConvertion(amount, currency, this.data.down.currency, "up");
        this.getDataChart(currency, this.data.down.currency);
      } else if (position === "down" && isSelect) {
        this.getConvertion(this.data.up.amount, this.data.up.currency, currency, "up");
        this.getDataChart(currency, this.data.up.currency);
      } else {
        const positionTemp = position == "up" ? "down" : "up";
        this.getConvertion(amount, currency, this.data[positionTemp].currency, position);
        if (position == this.data.result.large)
          this.getDataChart(currency, this.data[positionTemp].currency);
      }
    },

    // handleResize() {
    //   this.windowWidth = window.innerWidth;
    // },
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
    window.addEventListener("resize", () => (this.windowWidth = window.innerWidth));
    this.getCurrencies();
    this.getConvertion(1, "EUR", "USD", "up");
    this.getDataChart("EUR", "USD");
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
    padding: 20px 20px 50px;
  }

  #chart {
    border-radius: 10px;
    padding: 15px 0;
  }
}
</style>
