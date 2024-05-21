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
        up: { amount: 0, currency: "EUR" },
        down: { amount: 0, currency: "USD" },
        position: "up",
      },
      chart: {
        status: false,
        changed: false,
      },
    };
  },

  methods: {
    /* 
      viene richiamata la funzione getData in base: 
        - al tipo di input (number o select)
        - alla componente da cui parte la ricerca (position up o down)
        - alla componente precendentemente selezionata (position up o down)
    */
    search(value, position, inputNumber) {
      const prev = this.data.position;
      const next = position;
      const notNext = next == "up" ? "down" : "up";

      // tutti i casi in cui questa condizione, per cui l'input è di tipo number e le position prev e next sono uguali, è falsa allora la proprietà changed diventa true
      // nella funzione getData, a fine chiamata axios, il valore di chart status cambia in base a se changed è true o false
      if (!(inputNumber && prev == next)) this.chart.changed = true;

      // input number: parte la funzione passando il value dell'input e le currency in base alla posizione nuova
      if (inputNumber) {
        this.getData(value, this.data[next].currency, this.data[notNext].currency, next);
      }

      // input select in base a se le posizioni sono uguali (posizione nuova) o diverse (posizione precedente)
      if (!inputNumber) {
        if (next == prev) {
          this.getData(this.data[next].amount, value, this.data[notNext].currency, next);
        } else {
          this.getData(this.data[prev].amount, this.data[prev].currency, value, prev);
        }
      }
    },

    getData(amount, from, to, position) {
      axios
        .get(this.baseUrl + "latest", { params: { amount: amount, from: from, to: to } })
        .then((res) => {
          // creo due oggetti from e to in base ai valori from e to passati nella chaiamta
          const from = {};
          from["amount"] = res.data.amount;
          from["currency"] = res.data.base;

          const to = {};
          for (const [key, value] of Object.entries(res.data.rates)) {
            to["amount"] = value;
            to["currency"] = key;
          }

          // in base al parametro position viene deciso quale valore assegnare alle proprietà up e down
          this.data.up = position === "up" ? from : to;
          this.data.down = position === "up" ? to : from;

          this.data.position = position;

          // se il grafico va modificato allora a fine chiamata axios la variabile status cambia valore
          if (this.chart.changed) {
            this.chart.status = !this.chart.status;
            this.chart.changed = false;
          }
        });
    },

    getCurrencies() {
      // ottengo la lista di monete da stampare nella select
      axios.get(this.baseUrl + "currencies").then((res) => {
        this.currencies = res.data;
      });
    },
  },

  mounted() {
    this.getCurrencies();
    this.getData(1, "EUR", "USD", "up");
  },
};
</script>

<template>
  <div class="my-container">
    <h1 class="text-center mb-3 mb-md-2">Currency Converter</h1>

    <Result :data="this.data" />

    <div class="d-flex flex-column gap-4 gap-md-3">
      <Search
        v-for="n in ['up', 'down']"
        :position="n"
        :data="this.data"
        :currencies="this.currencies"
        @search="search"
      />
    </div>

    <Chart :data="this.data" :status="this.chart.status" />
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
