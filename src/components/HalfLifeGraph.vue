<script>
import { Line, mixins } from "vue-chartjs";
const { reactiveData } = mixins;
import { range, sum } from "lodash";

export default {
  extends: Line,
  mixins: [reactiveData],
  data() {
    return {
      gradients: [],
      hours: 10,
      options: {
        maintainAspectRatio: false,
        responsive: true,
        duration: 200,
        defaultColor: "#FF41B4"
      }
    };
  },
  props: ["graphOptions"],
  methods: {
    makeDataset: function(dataset, i, overrides = {}) {
      return {
        label: dataset.label,
        borderColor: "rgba(0, 231, 255, 0.9)",
        pointBackgroundColor: "white",
        pointBorderColor: "#ccc",
        backgroundColor: this.gradients[1],
        borderWidth: 1,
        data: dataset.data,
        cubicInterpolationMode: "monotone",
        order: i,
        ...overrides
      };
    },
    makeGradients: function() {
      if (this.gradients.length) return;

      this.gradients = [
        this.$refs.canvas.getContext("2d").createLinearGradient(0, 0, 0, 450),
        this.$refs.canvas.getContext("2d").createLinearGradient(0, 0, 0, 450)
      ];

      this.gradients[0].addColorStop(0, "rgba(255, 0,0, 0.5)");
      this.gradients[0].addColorStop(0.5, "rgba(255, 0, 0, 0.25)");
      this.gradients[0].addColorStop(1, "rgba(255, 0, 0, 0)");

      this.gradients[1].addColorStop(0, "rgba(0, 231, 255, 0.9)");
      this.gradients[1].addColorStop(0.5, "rgba(0, 231, 255, 0.25)");
      this.gradients[1].addColorStop(1, "rgba(0, 231, 255, 0)");
    },
    setChartData() {
      const data = {
        labels: this.hoursAxis,
        datasets: this.datasets.map((dataset, i) =>
          this.makeDataset(dataset, i)
        )
      };

      const totalDataset = this.makeDataset(
        {
          label: "Total",
          data: this.hoursAxis.map((_, i) =>
            sum(data.datasets.map(ds => ds.data[i]))
          )
        },
        -1,
        {
          borderWidth: 2,
          borderColor: "rgba(255, 0,0, 0.9)",
          backgroundColor: this.gradients[0]
        }
      );

      data.datasets.unshift(totalDataset);

      this.chartData = data;
    }
  },
  watch: {
    graphOptions: {
      handler: function(graphOptions) {
        this.hours = graphOptions.hours;
        this.setChartData();
      }
    }
  },
  computed: {
    hoursAxis() {
      const step = 0.5;

      return range(0, this.hours + step, step);
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
  },
  mounted() {
    this.makeGradients();
    this.setChartData();

    this.renderChart(this.chartData, this.options);
  }
};
</script>
