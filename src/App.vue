<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);

const weightChartEl = ref(null);

const weightChart = shallowRef(null);

const weightInput = ref(0);

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime(),
  });
};

watch(
  weights,
  (newWeights) => {
    const ws = [...newWeights];

    if (weightChart.value) {
      weightChart.value.data.labels = ws
        .sort((a, b) => a.date - b.date)
        .map((weight) => new Date(weight.date).toLocaleDateString())
        .slice(-7);

      weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map((weight) => weight.weight)
        .slice(-7);

      weightChart.value.update();
      return;
    }

    nextTick(() => {
      weightChart.value = new Chart(weightChartEl.value.getContext("2d"), {
        type: "line",
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map((weight) => new Date(weight.date).toLocaleDateString())
            .slice(-7),
          datasets: [
            {
              label: "Weight",
              data: ws
                .sort((a, b) => a.date - b.date)
                .map((weight) => weight.weight)
                .slice(-7),
              backgroundColor: "rgba(5, 12, 156,0.2)",
              borderColor: "#050C9C",
              borderWidth: 1,
              fill: true,
            },
          ],
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
        },
      });
    });
  },
  { deep: true }
);
</script>

<template>
  <main
    class="bg-blue-300 min-h-screen flex flex-col items-center justify-center"
  >
    <h1 class="text-3xl font-bold mb-6 text-blue-900">Weight Tracker</h1>

    <div
      class="current bg-white rounded-full shadow-xl border-4 border-blue-800 p-6 mb-6"
    >
      <span class="block text-3xl font-bold text-blue-900 text-center">{{
        currentWeight.weight
      }}</span>
      <small class="block text-blue-500 italic">Current Weight (kg)</small>
    </div>

    <form
      @submit.prevent="addWeight"
      class="flex mb-6 w-full max-w-md border border-blue-500 rounded-lg overflow-hidden transition duration-200 hover:border-blue-900 focus-within:border-blue-900"
    >
      <input
        type="number"
        step="0.1"
        v-model="weightInput"
        class="input-field flex-1 py-4 px-6 bg-blue-100 text-blue-900 outline-none"
        placeholder="Enter weight..."
      />

      <input
        type="submit"
        value="Add weight"
        class="submit-button bg-blue-900 text-white py-2 px-4 cursor-pointer text-lg font-semibold transition duration-200 ml-2"
      />
    </form>

    <div v-if="weights.length > 0" class="w-full max-w-xl">
      <h2 class="text-2xl font-semibold mb-4 text-blue-900">Last 7 days</h2>

      <div class="canvas-box bg-white shadow-xl p-4 rounded-lg mb-6">
        <canvas ref="weightChartEl"></canvas>
      </div>

      <div class="weight-history">
        <h2 class="text-2xl font-semibold mb-4 text-blue-900">
          Weight History
        </h2>
        <ul class="bg-blue-500 rounded-lg">
          <li
            v-for="weight in weights"
            :key="weight.date"
            class="flex justify-between items-center p-4 cursor-pointer hover:bg-white"
          >
            <span class="text-2xl font-semibold text-blue-900"
              >{{ weight.weight }}kg</span
            >
            <small class="text-blue-900">{{
              new Date(weight.date).toLocaleDateString()
            }}</small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "montserrat", sans-serif;
}

@media (max-width: 768px) {
  .canvas-box {
    max-width: 100%;
  }

  .weight-history {
    max-width: 100%;
  }

  .input-field {
    padding: 0.75rem 1rem;
    font-size: 1rem;
  }

  .submit-button {
    padding: 0.5rem 0rem;
    font-size: 1rem;
    text-align: center;
  }
}
</style>
