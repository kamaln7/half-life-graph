<template>
  <Graph :chartData="chartData" />
</template>

<script>
export default {
  props: {
    hours: Number
  },
  components: {
    Graph
  },
  computed: {
    hoursAxis() {
      const step = 0.5;

      return range(0, this.hours + step, step);
    },
    chartData() {
      const data = {
        labels: this.hoursAxis,
        datasets: this.datasets.map((dataset, i) => makeDataset(dataset, i))
      };

      const totalDataset = makeDataset(
        {
          label: "Total",
          data: this.hoursAxis.map((_, i) =>
            sum(data.datasets.map(ds => ds.data[i]))
          )
        },
        -1
      );

      data.datasets.unshift(totalDataset);

      return data;
    },
    datasets() {
      const d1 = {
        label: "First",
        data: this.hoursAxis.map(h => 10 * Math.pow(0.5, h / 3.5))
      };
      const d2 = {
        label: "Second",
        data: this.hoursAxis.map(h => {
          if (h < 3.5) return 0;

          return 10 * Math.pow(0.5, (h - 3.5) / 3.5);
        })
      };

      return [d1, d2];
    }
  }
};
</script>
