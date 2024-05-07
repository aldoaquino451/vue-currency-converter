<script>
import { state } from '../state.js';

export default {
  name: "Search",

  props: {
    currencies: Object,
    inputNumber: Number,
  },

  data() {
    return {
      state,

      currencyKey: '$',

    }
  },

  methods: {
    selected(key, inputNumber, from, to) {

      if (key === from && inputNumber === 1) {
        return true
      } else if (key === to && inputNumber === 2) {
        return true
      }
    },

    changeKey() {
      if (this.inputNumber === 1) {
        let key = document.querySelector(`#select-1`).value
        this.state.fromKey = key
      } else {
        let key = document.querySelector(`#select-2`).value
        this.state.toKey = key
      }

    },

    emitConvertion(inputNumber) {
      if (inputNumber === 1) {
        this.$emit('getConvertion', this.state.currencyToSearch_1, this.state.fromKey, this.state.toKey, inputNumber);
      } else {
        this.$emit('getConvertion', this.state.currencyToSearch_2, this.state.toKey, this.state.fromKey, inputNumber);
      }
    }
  }

}
</script>


<template>

  <div class="d-flex gap-3">
    <div class="input-group">

      <input type="number" class="form-control" step="0.00001" v-model="this.state.currencyToSearch_1"
        @keyup="emitConvertion(inputNumber)" v-if="this.inputNumber === 1">

      <input type="number" class="form-control" step="0.00001" v-model="this.state.currencyToSearch_2"
        @keyup="emitConvertion(inputNumber)" v-else>

      <span class="input-group-text">{{ currencyKey }}</span>
    </div>

    <select :id="`select-${inputNumber}`" class="form-select" @change="changeKey(), emitConvertion(inputNumber)">
      <option :value="key" v-for="(currency, key) in currencies"
        :selected="selected(key, this.inputNumber, this.state.fromKey, this.state.toKey)">
        <div>
          {{ currency }}
        </div>
      </option>
    </select>
  </div>

</template>


<style lang="scss" scoped></style>