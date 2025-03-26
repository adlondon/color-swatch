<script setup lang="ts">
const value = defineModel<number | undefined>({ required: false });

const handleInput = (event: Event) => {
  const input = event.target as HTMLInputElement;
  // A bug I'm stuck on is that typing any non numeric character will remove the entire value.
  // Please dont test me on regex. I will fail.
  input.value = input.value.replace(/\D/g, '');
};
</script>

<template>
  <input v-model.number="value" type="number" @input="handleInput" />
</template>

<style scoped>
/* Remove the up and down arrows from the native input because they ugly */
input {
  appearance: textfield;
}
input::-webkit-inner-spin-button,
input::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
</style>
