<script>
export default {
  name: "Search",

  props: {
    currencies: Object,
    position: String,
    data: Object,
  },

  computed: {
    currencyCode() {
      const currency =
        this.position === "up" ? this.data.up.currency : this.data.down.currency;

      const amountCurrency = new Intl.NumberFormat("it-IT", {
        style: "currency",
        currency: currency,
      });

      const result = amountCurrency.format(1);
      return result.split(/\s/g)[1];
    },

    disabled() {
      return this.position === "up" ? "down" : "up";
    },
  },
};
</script>

<template>
  <div class="d-flex gap-3 flex-column flex-md-row">
    <div class="input-group">
      <input
        v-model="data[position].amount"
        @keyup="$emit('search', data[position].amount, data[position].currency, position)"
        @change="
          $emit('search', data[position].amount, data[position].currency, position)
        "
        type="number"
        class="form-control"
      />
      <span class="input-group-text currencyCode d-flex justify-content-center">{{
        currencyCode
      }}</span>
    </div>

    <select
      v-model="data[position].currency"
      @change="$emit('search', data[position].amount, data[position].currency, position)"
      class="form-select"
    >
      <option
        :disabled="data[disabled].currency === short"
        :value="short"
        v-for="(long, short) in currencies"
      >
        {{ long }}
      </option>
    </select>
  </div>
</template>

<style lang="scss" scoped>
.currencyCode {
  width: 60px;
}
</style>
