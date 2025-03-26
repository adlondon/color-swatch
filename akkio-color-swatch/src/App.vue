<script setup lang="ts">
type Color = {
  name: string;
  hue: number;
  rgb: string;
};

import NumberInput from './components/NumberInput.vue';
import { ref } from 'vue';
const saturation = ref<number>(0);
const lightness = ref<number>(0);
const namedColors = ref<Color[]>([]);

// Assume this would live elsewhere with proper state management
const fetchData = async () => {
  const colorArray = [...Array(360).keys()];
  namedColors.value = [];
  try {
    colorArray.forEach(async (color) => {
      const response = await fetch(
        `https://www.thecolorapi.com/id?hsl=${color},${saturation.value}%,${lightness.value}%`,
      );
      const data = await response.json();

      // I assume that `exact_match_name` is always a boolean but who knows...
      if (data.name.exact_match_name === true) {
        namedColors.value.push({
          name: data.name.value,
          hue: data.hsl.h,
          rgb: data.rgb.value,
        });
      }
    });

    namedColors.value.sort((a, b) => a.name.localeCompare(b.name));
  } catch (error) {
    console.error('Error fetching data:', error);
  }
};
</script>

<template>
  <main>
    <NumberInput v-model="saturation" />
    <NumberInput v-model="lightness" />
    <button @click="fetchData">Fetch Colors</button>
    <div v-for="color in namedColors" :key="color.name">
      <div
        class="swatch"
        :style="{ backgroundColor: `hsl(${color.hue}, ${saturation}%, ${lightness}%)` }"
      />
      <div>{{ color.name }}</div>
      <div>{{ color.rgb }}</div>
    </div>
  </main>
</template>

<style scoped>
main {
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 1rem;
}

.swatch {
  width: 100px;
  height: 100px;
}
</style>
