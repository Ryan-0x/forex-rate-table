<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";

const apiData = ref({});
async function fetchData() {
  const getRequest = await axios.get("https://api.apilayer.com/fixer/latest", {
    headers: {
      apikey: "9sOx2d1gWYZrKY0uD4hCbOMIQzLcL4KQ",
    },
  });
  apiData.value = getRequest.data;
  // console.log(apiData);
}

function getDecimalPlaces(num) {
  const match = ("" + num).match(/(?:\.(\d+))?(?:[eE]([+-]?\d+))?$/);
  if (!match) {
    return 0;
  }
  return Math.max(
    0,
    (match[1] ? match[1].length : 0) - (match[2] ? +match[2] : 0)
  );
}

function formatValue(value, addition) {
  const decimalPlaces = getDecimalPlaces(value);
  const result = value + addition;
  // console.log(addition);
  return result.toFixed(decimalPlaces);
}
function getClass(key, value) {
  return key === "HKD" || parseInt(value) % 2 === 0
    ? "border-red-700 border-2"
    : "";
}
onMounted(() => {
  fetchData();
});
</script>

<template>
  <header></header>
  <div>
    <h1>Date: {{ apiData.date }}</h1>
  </div>

  <table class="table-auto border-collapse border-2">
    <thead>
      <tr>
        <th>Currency</th>
        <th>Rate</th>
        <th>New Rate</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(value, key) in apiData.rates" :key="key">
        <td :class="getClass(key, 1)">{{ key }}</td>
        <td :class="getClass(key, value)">
          {{ value }}
        </td>
        <td :class="getClass(key, formatValue(value, 10.0002))">
          {{ formatValue(value, 10.0002) }}
        </td>
      </tr>
    </tbody>
  </table>
</template>

<style scoped>
html,
body,
#app {
  height: 100%;
  background: white;
}

header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
