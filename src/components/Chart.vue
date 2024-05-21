<script>
import ApexCharts from "apexcharts";
import axios from "axios";
import "@fontsource-variable/rubik";

export default {
  name: "Chart",

  props: {
    status: Boolean,
    data: Object,
  },

  data() {
    return {
      windowWidth: null,
      chartDOM: null,
      chart: { from: "", to: "", dates: [], values: [] },
    };
  },

  methods: {
    getChart(data) {
      // l'interavallo di date sarà lungo 7 giorni in versione mobile e 1 mese in versione desktop/tablet
      const period =
        this.windowWidth >= 768 ? " nell'ultimo mese" : " nell'ultima settimana";

      // opzioni grafico
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
          fontFamily: "Rubik Variable, sans-serif",
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
          text: "Rapporto fra " + data.from + " e " + data.to + period,
          align: "center",
          margin: 60,
          offsetX: 0,
          offsetY: 0,
          floating: true,
          style: {
            fontSize: "14px",
            fontWeight: "bold",
            fontFamily: "Rubik Variable, sans-serif",
            color: "#d4d4d4",
          },
        },
        xaxis: {
          categories: data.dates,
        },
      };

      // svuoto l'elemento #chart e lo uso per renderizzare un nuovo grafico
      this.chartDOM.innerHTML = null;
      const chartObj = new ApexCharts(this.chartDOM, options);
      chartObj.render();
    },

    getDataChart(from, to) {
      this.chart.from = from;
      this.chart.to = to;

      // l'interavallo di date sarà lungo 7 giorni in versione mobile e 1 mese in versione desktop/tablet
      const now = new Date();
      if (this.windowWidth >= 768) now.setMonth(now.getMonth() - 1);
      else now.setDate(now.getDate() - 7);
      let date = now.toLocaleDateString("us-US").split("/").reverse();
      date = date.map((el) => el.padStart(2, "0")).join("-");

      // tramite chiamata API ottengo la valuta from e to e una lista di date e di quantità
      // a fine chiamata axios parte la funzione getChart()
      axios
        .get("https://api.frankfurter.app/" + date + "..", {
          params: { from: from, to: to },
        })
        .then((res) => {
          this.chart.dates = [];
          this.chart.values = [];

          const rates = Object.entries(res.data.rates);
          rates.forEach((rate) => {
            this.chart.values.push(rate[1][to]);

            let date = new Date(rate[0]);
            date = date.toLocaleDateString("it-IT", {
              month: "short",
              day: "numeric",
            });
            this.chart.dates.push(date);
          });

          this.getChart(this.chart);
        });
    },
  },

  watch: {
    // nella componente App la variabile status cambia valore ogni volta che bisogna modificare il grafico
    status() {
      let from, to;

      if (this.data.position == "up")
        [from, to] = [this.data.up.currency, this.data.down.currency];
      else [from, to] = [this.data.down.currency, this.data.up.currency];

      this.getDataChart(from, to);
    },

    // se la viewport viene ridimensionata (si scende sotto o si sale sopra il valore 768) il grafico viene modificato
    windowWidth(next, prev) {
      if ((prev < 768 && next >= 768) || (prev >= 768 && next < 768)) {
        this.getDataChart(this.chart.from, this.chart.to);
      }
    },
  },

  mounted() {
    // la finesta rimane in ascolto di un ridimensionamento
    window.addEventListener("resize", () => (this.windowWidth = window.innerWidth));

    this.windowWidth = window.innerWidth;
    this.chartDOM = document.querySelector("#chart");
    this.getDataChart("EUR", "USD");
  },
};
</script>

<template>
  <div id="chart"></div>
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

#chart text {
  color: black;
}

@media screen and (max-width: 767px) {
  #chart {
    border-radius: 10px;
    padding: 15px 0;
  }
}
</style>
