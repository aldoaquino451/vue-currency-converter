<script>
export default {
  name: "Result",

  props: {
    data: Object,
  },

  methods: {
    // viene passata la posizione up o down e viene restituito un valore formattato: amount (due decimali) e currency (simbolo o acronimo)
    toFormat(position) {
      const obj = this.data[position];
      const temp = new Intl.NumberFormat("it-IT", {
        style: "currency",
        currency: obj.currency,
      });
      const formatted = temp.format(obj.amount);
      return formatted;
    },
  },

  computed: {
    // ricerca: amount e currency che provengono da from
    query() {
      const position = this.data.position;
      return this.toFormat(position);
    },

    // risultato: amount e currency che provengono da to
    response() {
      const notPosition = this.data.position == "up" ? "down" : "up";
      return this.toFormat(notPosition);
    },
  },
};
</script>

<template>
  <div class="mb-3 mb-md-4">
    <p class="small mb-0 mb-md-1">{{ query }} corrisponde a:</p>
    <p class="large">{{ response }}</p>
  </div>
</template>

<style lang="scss" scoped>
.small {
  font-size: 1em;
}

.large {
  font-size: 1.8em;
}
</style>
