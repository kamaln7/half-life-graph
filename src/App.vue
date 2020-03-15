<template>
  <div id="app" class="mw8 center ma2 sans-serif mt4 ph2">
    <header>
      <h1 class="f4">Half-life Graph</h1>
    </header>

    <div class="black-80 mb3">
      <div class="measure-narrow">
        <label for="hours" class="f6 b db mb2"
          >Showing activity over
          <span class="gold bg-near-black br1 ph1">{{ this.hours }}</span>
          hours.</label
        >
        <div class="flex items-center">
          <span>1</span>
          <input
            v-model="hours"
            id="hours"
            type="range"
            min="1"
            max="24"
            step="0.5"
            value="10"
            class="mh1 w-100"
          />
          <span>24</span>
        </div>
      </div>
    </div>

    <div class="bg-near-black pink br3 pa2">
      <HalfLifeGraph
        :graphOptions="graphOptions"
        :styles="{ height: '28rem', position: 'relative', width: '100%' }"
      />
    </div>
  </div>
</template>

<script>
import HalfLifeGraph from "@/components/HalfLifeGraph.vue";

export default {
  name: "App",
  components: { HalfLifeGraph },
  data() {
    return {
      hours: 10
    };
  },
  computed: {
    graphOptions() {
      return {
        hours: parseInt(this.hours)
      };
    }
  }
};
</script>
