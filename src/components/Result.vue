<script>
export default {
  name: "Result",
  props: {
    data: Object,
  },
  computed: {
    small() {
      return this.data.result.small;
    },
    large() {
      return this.data.result.large;
    },
  },
};
</script>
<template>
  <div class="mb-4">
    <p class="small">
      <span> {{ data[small].amount }} {{ data[small].currency }} </span>
      è uguale a:
    </p>
    <p class="large">{{ data[large].amount }} {{ data[large].currency }}</p>
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
