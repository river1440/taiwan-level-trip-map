<script setup>
import TaiwanMap from './components/TaiwanMap.vue'
import BaseFooter from './components/BaseFooter.vue'
</script>

<template>
  <div class="min-h-screen flex flex-col">
    <main class="p-4 flex-grow">
      <h1 class="text-3xl font-bold text-center mb-4">TAIWAN LEVEL UP ✨</h1>
      <TaiwanMap />
    </main>
    <BaseFooter />
  </div>
</template>
