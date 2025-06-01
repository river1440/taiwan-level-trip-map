<template>
  <div class="p-4">
    <h1 class="text-2xl font-bold mb-4">Taiwan Visit Tracker</h1>

    <div
      class="svg-container max-w-3xl"
      v-html="svgContent"
      @click="handleRegionClick"
    ></div>

    <div class="mt-6">
      <h2 class="text-lg font-semibold">Visit Counts:</h2>
      <ul class="list-disc list-inside">
        <li v-for="city in visitData">
          {{ city.title }}: {{ city.status}}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, reactive } from 'vue'

const svgContent = ref('')
const visitData = reactive<Record<string, { id: string; title: string; status: number }>>({})

onMounted(async () => {
  const res = await fetch('/maps/taiwan_map.svg')
  const text = await res.text()
  svgContent.value = text

  // Initialize visit data from SVG IDs
  const parser = new DOMParser()
  const doc = parser.parseFromString(text, 'image/svg+xml')
  const countyGroup = doc.querySelector('g#county')
  
  if (countyGroup) {
    const paths = countyGroup.querySelectorAll('path[id]')

    paths.forEach((el) => {
      const id = el.id
      const titleEl = el.querySelector('title')
      const title = titleEl?.textContent?.trim() || id

      visitData[id] = {
        id,
        title,
        status: 0,
      }
    })
  }
})

function handleRegionClick(event: MouseEvent) {
  const target = event.target as HTMLElement
  const regionId = target?.id
  if (regionId && visitData[regionId] !== undefined) {
    visitData[regionId].status++

    // Change color directly (basic version — you can enhance with Vue bindings)
    target.setAttribute('fill', getColor(visitData[regionId].status))
  }
}

function getColor(status: number) {
  if (status>= 5) return '#f97316' // orange
  if (status>= 1) return '#60a5fa' // blue
  return '#e5e7eb' // gray
}

</script>