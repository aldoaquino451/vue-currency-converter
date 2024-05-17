<script>
export default {
  name: "Search",

  data() {
    return {
      amount: "",
    };
  },
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

    isActive() {
      return this.data.result.small == this.position;
    },
  },

  watch: {
    amount(newValue, oldValue) {
      console.log("changed");
    },
  },
};
</script>

<template>
  <div class="d-flex gap-2 gap-md-3 flex-column flex-md-row">
    <div class="input-group input-box" :class="isActive ? 'active' : ''">
      <input
        v-model="data[position].amount"
        @keyup="
          $emit('search', data[position].amount, data[position].currency, position, false)
        "
        type="number"
        class="form-control"
        step="0.0001"
        min="0.0001"
      />
      <span class="input-group-text currencyCode d-flex justify-content-center">{{
        currencyCode
      }}</span>
    </div>

    <select
      v-model="data[position].currency"
      @change="
        $emit('search', data[position].amount, data[position].currency, position, true)
      "
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

.active {
  box-shadow: 0px 0px 3px 3px rgba(255, 255, 255, 0.4);
}
</style>
