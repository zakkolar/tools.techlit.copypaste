<script setup>
defineProps(['question'])
</script>

<template>
  <details class="text-xl/9 [&_a]:underline">
    <summary class="font-bold">{{ question }}</summary>
    <p><slot></slot></p>
  </details>
</template>
