<script>
export default {
  name: "Search",

  props: {
    currencies: Object,
    position: String,
    data: Object,
  },

  computed: {
    disabled() {
      return this.position === "up" ? "down" : "up";
    },
  },
};
</script>

<template>
  <div class="d-flex gap-3">
    <div class="input-group">
      <input
        v-model="data[position].amount"
        @keyup="$emit('search', data[position].amount, data[position].currency, position)"
        @change="
          $emit('search', data[position].amount, data[position].currency, position)
        "
        type="number"
        step="0.0001"
        min="0.0001"
        class="form-control"
      />
      <span class="input-group-text">$</span>
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

<style lang="scss" scoped></style>
