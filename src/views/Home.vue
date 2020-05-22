<template lang="pug">
	div
		div(v-show="online")
			div(:class="classes") {{ ping }} ms
			div
				br
				br
			div.chart-container
				canvas.chart.darkmode-ignore(ref="chart" height="400")
		div(v-show="!online")
			div
				img.offline-icon(:src="'/img/offline.svg'" alt="No connection")
			p.text.offline-text No connection available
</template>
<script lang="ts">
import Chart from "chart.js";
import Darkmode from "darkmode-js";
import Vue from "vue";

export default Vue.extend({
  data() {
    return {
      ping: 0,
      chart: null,
      historyRetention: 10,
      timeout: null,
      online: true,
      offlineImagePath:
        (process.env.NODE_ENV === "production" ? "/ping/" : "/") +
        "img/offline.svg",
    };
  },
  computed: {
    quality() {
      let quality = "good";

      if (this.ping > 500) {
        quality = "bad";
      } else if (this.ping > 100) {
        quality = "medium";
      }

      return quality;
    },
    classes() {
      return `ping ${this.quality} text darkmode-ignore`;
    },
  },
  async created() {
    await this.setPing();
  },
  methods: {
    async setPing() {
      if (this.timeout) {
        clearTimeout(this.timeout);
      }

      if (!this.online) {
        this.timeout = setTimeout(async () => await this.setPing(), 5000);
      } else {
        const start = window.performance.now();
        let response = null;
        try {
          response = await fetch("https://8.8.8.8", { mode: "no-cors" });
        } catch (exception) {
          const durationInMilliseconds = window.performance.now() - start;

          if (durationInMilliseconds < 5000) {
            this.timeout = setTimeout(
              async () => await this.setPing(),
              5000 - durationInMilliseconds
            );
          } else {
            await this.setPing();
          }

          return;
        }
        const durationInMilliseconds = window.performance.now() - start;
        this.ping = Math.round(durationInMilliseconds);
        this.addChartValue();

        if (durationInMilliseconds < 5000) {
          this.timeout = setTimeout(
            async () => await this.setPing(),
            5000 - durationInMilliseconds
          );
        } else {
          await this.setPing();
        }
      }
    },
    addChartValue() {
      this.chart.data.labels.push(this.ping);

      if (this.chart.data.labels.length > this.historyRetention) {
        this.chart.data.labels.shift();
      }

      for (const dataset of this.chart.data.datasets) {
        dataset.data.push(this.ping);

        let color = "rgba(0, 204, 153, .6)";

        if (this.quality === "medium") {
          color = "rgba(255, 153, 102, .6)";
        } else if (this.quality === "bad") {
          color = "rgba(255, 0, 102, .6)";
        }

        dataset.backgroundColor = Array(dataset.backgroundColor.length).fill(
          color
        );
        dataset.backgroundColor.push(color);

        if (dataset.data.length > this.historyRetention) {
          dataset.data.shift();
          dataset.backgroundColor.shift();
        }
      }

      this.chart.update();
    },
  },
  mounted() {
    this.chart = new Chart(this.$refs.chart, {
      type: "line",
      options: {
        responsive: false,
        maintainAspectRatio: false,
        legend: {
          display: false,
        },
      },
      data: {
        labels: [0],
        datasets: [
          {
            data: [0],
            backgroundColor: ["rgba(0, 204, 153, .6)"],
          },
        ],
      },
    });

    new Darkmode({
      label: "🌓",
    }).showWidget();

    window.addEventListener("offline", () => (this.online = false));
    window.addEventListener("online", () => (this.online = true));

    this.online = navigator.onLine;
  },
});
</script>

<style lang="sass">
@import url('https://fonts.googleapis.com/css2?family=Fira+Code&display=swap')

body
	margin: 0

.ping
	font-weight: bold
	font-size: 6vw
	color: lightgrey

	&.good
		color: #00cc99

	&.medium
		color: #ff9966

	&.bad
		color: #ff0066

.text
	font-family: 'Fira Code', monospace

.chart-container
	width: 80vw
	height: 40vh

.chart
	width: 80vw
	margin-left: 10vw
	margin-right: 10vw

.darkmode-toggle
	z-index: 1

.offline-icon
	width: 30vw
	height: auto

.offline-text
	font-size: 3vw
</style>
