<script>
import ApexCharts from "apexcharts";

export default {
  name: "Test",

  props: {
    dataChart: Object,
    chartFromTo: String,
  },

  methods: {
    getChart(data) {
      console.log(data);
      console.log("--------------- ok");
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
            data: data.values,
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
          text: "Rapporto fra " + data.from + " e " + data.to + " nell'ultimo mese",
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
          categories: data.dates,
        },
      };
      const chartObj = new ApexCharts(chart, options);
      chartObj.render();
    },
  },

  watch: {
    chartFromTo(newValue, oldValue) {
      this.getChart(this.dataChart);
    },
  },
};
</script>

<template>
  <div>
    <p>test</p>
    <p>{{ chartFromTo }}</p>
    <p>{{ dataChart }}</p>
    <div id="chart"></div>
  </div>
</template>

<style lang="scss" scoped>
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
  #chart {
    border-radius: 10px;
    padding: 15px 0;
  }
}
</style>
