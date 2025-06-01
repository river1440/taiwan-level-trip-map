<template>
  <div class="grid gap-2">
    <div
      v-for="(label, index) in levels"
      :key="index"
      class="flex items-center space-x-2 cursor-pointer p-2 rounded hover:bg-gray-100"
      @click="$emit('select', index)"
    >
      <div :class="['w-4 h-4 rounded-full', levelColors[index]]"></div>
      <span>{{ index }}: {{ label }}</span>
    </div>
  </div>
</template>

<script setup>
const levelColors = [
  'bg-white border',
  'bg-blue-500',
  'bg-green-500',
  'bg-yellow-400',
  'bg-red-500',
  'bg-pink-400'
]

const levels = ['未踏', '通過', '接地', '訪問', '宿泊', '居住']
</script>