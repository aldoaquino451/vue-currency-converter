<script>
import ApexCharts from "apexcharts";

export default {
  name: "Chart",

  props: {
    data: Object,
  },

  methods: {
    getChart() {
      var options = {
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
            data: this.data.values,
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
        xaxis: {
          categories: this.data.dates,
        },
      };

      var chart = new ApexCharts(document.querySelector("#chart"), options);

      chart.render();
    },
  },

  mounted() {
    setTimeout(() => {
      this.getChart();
    }, 700);
  },
};
</script>

<template>
  <div id="chart"></div>
</template>

<style lang="scss" scoped>
#chart {
  margin: 50px auto;
  margin-bottom: 0;
}
</style>
