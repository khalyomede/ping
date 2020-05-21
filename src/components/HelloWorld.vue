<template lang="pug">
	div {{ ping }}
</template>

<script lang="ts">
import Vue from "vue";

export default Vue.extend({
  name: "HelloWorld",
  data() {
    return {
      ping: 0,
    };
  },
  async created() {
    await this.setPing();
  },
  methods: {
    async setPing() {
      const start = window.performance.now();
      const response = await fetch("https://8.8.8.8", { mode: "no-cors" });
      const durationInMilliseconds = window.performance.now() - start;
      this.ping = Math.round(durationInMilliseconds);

      if (durationInMilliseconds < 5000) {
        setTimeout(
          async () => await this.setPing(),
          5000 - durationInMilliseconds
        );
      } else {
        await this.setPing();
      }
    },
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="sass"></style>
