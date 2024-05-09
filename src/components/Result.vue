<script>
export default {
  name: "Result",
  props: {
    data: Object,
  },
  computed: {
    small() {
      const small = this.data.result.small;

      const amountCurrency = new Intl.NumberFormat("it-IT", {
        style: "currency",
        currency: this.data[small].currency,
      });

      return amountCurrency.format(this.data[small].amount);
    },

    large() {
      const large = this.data.result.large;

      const amountCurrency = new Intl.NumberFormat("it-IT", {
        style: "currency",
        currency: this.data[large].currency,
      });

      return amountCurrency.format(this.data[large].amount);
    },
  },
};
</script>
<template>
  <div class="mb-4">
    <p class="small">
      <span> {{ small }} </span>
      è uguale a:
    </p>
    <p class="large">{{ large }}</p>
  </div>
</template>
<style lang="scss" scoped>
.small {
  font-size: 1em;
}

.large {
  font-size: 1.6em;
}
</style>
