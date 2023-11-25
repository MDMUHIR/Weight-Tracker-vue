<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);

const weightChartEl = ref(null);

const weightChart = shallowRef(null);

const weightInput = ref();

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
        .map((w) => new Date(w.date).toLocaleDateString())
        .splice(-7);

      weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map((w) => w.weight)
        .splice(-7);

      weightChart.value.update();
      return;
    }

    nextTick(() => {
      weightChart.value = new Chart(weightChartEl.value.getContext("2d"), {
        type: "line",
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map((w) => new Date(w.date).toLocaleDateString()),
          datasets: [
            {
              label: "weight",
              data: ws.sort((a, b) => a.date - b.date).map((w) => w.weight),
              backgroundColor: "rgba(255, 105, 180, 0.2)",
              borderColor: "rgb(255, 105, 180)",
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
      console.log(weightChartEl);
    });
  },
  { deep: true }
);
</script>

<template>
  <div class="container-sm d-flex flex-column align-items-center">
    <h2
      class="mt-4 mb-4 p-3 rounded border border-danger border-opacity-25 shadow"
    >
     🙅🏾 Weight Tracker 🙅🏼‍♂️
     
    </h2>

    <div
      class="currentWeight rounded rounded-circle d-flex flex-column align-items-center justify-content-center border border-danger border-4 shadow-lg bg-light mb-4"
      style="width: 200px; height: 200px"
    >
      <p class="fs-1 fw-bold mb-0">{{ currentWeight.weight }}</p>
      <small class="fst-italic">Current weight (kg)</small>
    </div>
    <form @submit.prevent="addWeight" class="w-100">
      <div class="mb-3 d-flex">
        <input
          type="number"
          step="0.1"
          v-model="weightInput"
          class="form-control rounded-0 rounded-start border-2 w-100"
          id="weight"
          placeholder="Enter Weight"
        />
        <button type="submit" class="btn btn-danger rounded-0 rounded-end">
          Add weight
        </button>
      </div>
    </form>
    <div v-if="weights && weights.length > 0" class="show-weight mt-3 w-100">
      <h2 class="text-warning mb-3 text-warning-emphasis">Last 7 Days</h2>
      <div class="canvas-box bg-light p-3 rounded w-100">
        <canvas ref="weightChartEl"></canvas>
      </div>
      <div class="history-box mt-4">
        <h2 class="text-danger-emphasis mb-3">Weight History</h2>
        <ul class="list-group">
          <li v-for="weight in weights" class="list-group-item  w-100">
            <span class="fw-semibold fs-5">{{ weight.weight }} kg</span>
            <small class="float-end mt-1">
              {{ new Date(weight.date).toLocaleDateString() }}
            </small>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<style scoped>
.form-control:focus {
  border-color: #dc3545;
  box-shadow: none;
}
</style>
