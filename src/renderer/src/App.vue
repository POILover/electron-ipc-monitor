<template>
  <button @click="handleClick">Click Me</button>
</template>
<script setup lang="ts">
const { randomPromise } = window.api

async function handleClick() {
  randomPromise()
}
</script>
