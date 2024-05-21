<script>
export default {
  name: "Search",

  props: {
    position: String,
    currencies: Object,
    data: Object,
  },

  computed: {
    // stampa il simbolo della moneta o, se non presente, una stringa di 3 lettere
    currencyCode() {
      const temp = new Intl.NumberFormat("it-IT", {
        style: "currency",
        currency: this.data[this.position].currency,
      });
      const formatted = temp.format(1);
      const code = formatted.split(/\s/g)[1];
      return code;
    },

    // permette di stabilire quale moneta è stata selezionata nell'input A e di disabilitarla nell'input B
    isDisabled() {
      return this.position == "up" ? this.data.down.currency : this.data.up.currency;
    },

    // applica la classe active all'input di tipo number da cui parte la ricerca
    isActive() {
      return this.data.position == this.position;
    },
  },
};
</script>

<template>
  <div class="d-flex gap-2 gap-md-3 flex-column flex-md-row">
    <div class="input-group input-box" :class="isActive ? 'active' : ''">
      <input
        v-model="data[position].amount"
        @keyup="$emit('search', data[position].amount, position, true)"
        type="number"
        class="form-control"
        step="0.0001"
        min="0.0001"
      />
      <span class="input-group-text currency-code d-flex justify-content-center">
        {{ currencyCode }}
      </span>
    </div>

    <select
      v-model="data[position].currency"
      @change="$emit('search', data[position].currency, position, false)"
      class="form-select"
    >
      <option
        :disabled="isDisabled == short"
        :value="short"
        v-for="(long, short) in currencies"
      >
        {{ long }}
      </option>
    </select>
  </div>
</template>

<style lang="scss" scoped>
.active {
  box-shadow: 0px 0px 3px 3px rgba(255, 255, 255, 0.4);
}

.currency-code {
  width: 60px;
}

.input-box {
  border-radius: 4px;
  overflow: hidden;
}

input:focus,
select:focus {
  box-shadow: none;
  border: none;
}

input:focus-visible,
select:focus-visible {
  outline: none;
}
</style>
