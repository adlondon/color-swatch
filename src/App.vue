<script setup lang="ts">
import { ref } from 'vue';
import type { Color } from './types';

import NumberInput from './components/NumberInput.vue';

const saturation = ref<number | undefined>(undefined);
const lightness = ref<number | undefined>(undefined);
const namedColors = ref<Color[]>([]);
const loading = ref<boolean>(false);
const errorMessage = ref<string | undefined>(undefined);
const cachedColors = ref<Color[]>([]);

const validateInput = () => {
  errorMessage.value = undefined;

  if (!saturation.value) {
    errorMessage.value = 'Saturation cannot be empty';
    return false;
  } else if (!lightness.value) {
    errorMessage.value = 'Lightness cannot be empty';
    return false;
  } else if (saturation.value > 100) {
    errorMessage.value = "Saturation can't be greater than 100";
    return false;
  } else if (lightness.value > 100) {
    errorMessage.value = "Lightness can't be greater than 100";
    return false;
  }

  return true;
};

const isColorCached = (hue: number) => {
  return cachedColors.value.some(
    (color) =>
      color.saturation === saturation.value &&
      color.lightness === lightness.value &&
      color.hue === hue,
  );
};

const getCachedColor = (hue: number): Color | undefined => {
  return cachedColors.value.find(
    (color) =>
      color.saturation === saturation.value &&
      color.lightness === lightness.value &&
      color.hue === hue,
  );
};

// Assume this would live elsewhere with proper state management
const fetchData = async () => {
  if (!validateInput()) {
    return;
  }

  namedColors.value = [];
  loading.value = true;
  try {
    const hueArray = [...Array(360).keys()];
    const colorPromises = hueArray.map(async (hue) => {
      if (isColorCached(hue)) {
        return getCachedColor(hue);
      }
      const response = await fetch(
        `https://www.thecolorapi.com/id?hsl=${hue},${saturation.value}%,${lightness.value}%`,
      );
      const data = await response.json();

      // Add colors to the cache
      const color: Color = {
        name: data.name.value,
        hue: data.hsl.h,
        saturation: data.hsl.s,
        lightness: data.hsl.l,
        rgb: data.rgb.value,
      };

      cachedColors.value.push(color);
      return color;
    });

    const results = await Promise.all(colorPromises);

    // Filter out dupes while preserving order
    namedColors.value = results
      .filter((color): color is Color => color !== undefined)
      .filter((color, index, self) => index === self.findIndex((c) => c.name === color.name));
  } catch (error) {
    console.error('Error fetching data:', error);
  } finally {
    loading.value = false;
  }
};
</script>

<template>
  <main>
    <form @submit.prevent="fetchData" class="swatch__controls">
      <NumberInput v-model="saturation" placeholder="Saturation" />
      <NumberInput v-model="lightness" placeholder="Lightness" />
      <button type="submit">Fetch Colors</button>
    </form>

    <p v-if="errorMessage" class="swatch__error">
      {{ errorMessage }}
    </p>

    <p v-if="loading">Loading...</p>

    <div v-else class="swatch__container">
      <div class="swatch__item" v-for="color in namedColors" :key="color.name">
        <div
          class="swatch__background"
          :style="{
            backgroundColor: `hsl(${color.hue}, ${color.saturation}%, ${color.lightness}%)`,
          }"
        />
        <p>{{ color.name }}</p>
        <p>{{ color.rgb }}</p>
      </div>
    </div>
  </main>
</template>

<style scoped>
main {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 1rem;
}

.swatch__error {
  color: red;
}

.swatch__controls {
  display: flex;
  gap: 1rem;
  width: 400px;
  flex-wrap: wrap;
  justify-content: center;
  height: 60px;

  @media (max-width: 400px) {
    width: 100%;
    flex-direction: column;
    flex-wrap: nowrap;
  }
}

.swatch__item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  width: 200px;
  height: 200px;
}

.swatch__background {
  width: 100px;
  height: 100px;
}

.swatch__container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}
</style>
