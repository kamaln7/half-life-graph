<script>
import Chart from "chart.js";
import { Line, mixins } from "vue-chartjs";
const { reactiveData } = mixins;
import { range, sum } from "lodash";

Chart.defaults.global.defaultFontColor = "#FFB700";
Chart.defaults.global.defaultFontStyle = "bold";
Chart.defaults.global.defaultFontFamily =
  "-apple-system,BlinkMacSystemFont,avenir next,avenir,helvetica neue,helvetica,ubuntu,roboto,noto,segoe ui,arial,sans-serif";
Chart.defaults.global.animation.duration = 400;

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
        scales: {
          yAxes: [
            {
              scaleLabel: {
                display: true,
                labelString: "Active (mg)"
              }
            }
          ],
          xAxes: [
            {
              scaleLabel: {
                display: true,
                labelString: "Hours"
              }
            }
          ]
        },
        tooltips: {
          callbacks: {
            label: function(tooltipItem, data) {
              var label = data.datasets[tooltipItem.datasetIndex].label || "";

              if (label) {
                label += ": ";
              }
              label += Math.round(tooltipItem.yLabel * 100) / 100;
              return label + "mg";
            }
          }
        }
      }
    };
  },
  props: ["graphOptions"],
  methods: {
    makeDataset: function(dataset, i, overrides = {}) {
      return {
        label: dataset.label,
        borderColor: "#FFFCEB",
        pointBackgroundColor: "#FFFCEB",
        pointBorderColor: "#FFFCEB",
        backgroundColor: this.gradients.common,
        borderWidth: 1.5,
        data: dataset.data,
        cubicInterpolationMode: "monotone",
        order: i,
        ...overrides
      };
    },
    makeGradients: function() {
      if (this.gradients.common) return;

      this.gradients = {
        common: this.$refs.canvas
          .getContext("2d")
          .createLinearGradient(0, 0, 0, 450),
        total: this.$refs.canvas
          .getContext("2d")
          .createLinearGradient(0, 0, 0, 450)
      };

      this.gradients.common.addColorStop(0, "rgba(255, 252, 235, 0.4)");
      this.gradients.common.addColorStop(0.5, "rgba(255, 252, 235, 0.2)");
      this.gradients.common.addColorStop(1, "rgba(255, 252, 235, 0.1)");

      this.gradients.total.addColorStop(0, "rgba(255, 183, 0, 0.9)");
      this.gradients.total.addColorStop(0.5, "rgba(255, 183, 0, 0.5)");
      this.gradients.total.addColorStop(1, "rgba(255, 183, 0, 0.1)");
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
          borderColor: "#FFB700",
          backgroundColor: this.gradients.total
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
