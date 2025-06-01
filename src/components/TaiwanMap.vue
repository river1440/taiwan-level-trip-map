<template>
  <div class="relative">
    <!-- <div ref="svgContainer" class="w-24 h-24"></div> -->
  <svg
    ref="svgContainer"
    viewBox="0 0 800 1000"
    preserveAspectRatio="xMidYMid meet"
    class="w-full h-full"
    xmlns="http://www.w3.org/2000/svg"
  >
  </svg>
  <div
    v-if="showPopover"
    class="popover"
    :style="{ top: `${popoverPos.y}px`, left: `${popoverPos.x}px` }"
  >
    <VisitPopover
      :selected="selectedLevel"
      @update:selected="updateVisitLevel"
    />
    </div>

    <div class="mt-4 p-4 bg-white shadow rounded">
      <h2 class="text-lg font-bold">總分：{{ totalScore }}</h2>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, computed } from 'vue'
import VisitPopover from './VisitPopover.vue'

const svgContainer = ref<HTMLElement | null>(null)
const op = ref(null)
const selectedRegion = ref(null)
const levels = [
  { level: 0, label: '未踏' },
  { level: 1, label: '通過' },
  { level: 2, label: '接地' },
  { level: 3, label: '訪問' },
  { level: 4, label: '宿泊' },
  { level: 5, label: '居住' },
]

const selectedLevel = ref(levels[0]) 

// Popover
const showPopover = ref(false)
const popoverPos = ref({ x: 0, y: 0 })

const visitData = reactive({})
const levelColors = ['#ffffff', '#3b82f6', '#22c55e', '#facc15', '#ef4444', '#ec4899']

const totalScore = computed(() => {
  return Object.values(visitData).reduce((sum, item) => sum + item.level, 0)
})

function onRegionClick(e) {
  const target = e.target
  selectedRegion.value = target.id
  selectedLevel.value = visitData[target.id].level

  popoverPos.value = {
    x: e.clientX, // offset a bit so popover is not exactly on cursor
    y: e.clientY,
  }

  showPopover.value = true
}

onMounted(async () => {
  const text = await fetch('/maps/taiwan_map.svg').then((res) => res.text())

  const parser = new DOMParser()
  const doc = parser.parseFromString(text, 'image/svg+xml')

  if (svgContainer.value) {
    svgContainer.value.innerHTML = ''
    svgContainer.value.appendChild(doc.documentElement) // insert parsed <svg>
  
    // Now query the appended SVG in the DOM, not in the detached doc!
    const countyGroup = svgContainer.value.querySelector('g#county')
    const paths = countyGroup?.querySelectorAll('path') || []

    paths.forEach((el) => {
      const id = el.id
      const title = el.querySelector('title')?.textContent || '未知'
      visitData[id] = { title, level: levels[0] }

      el.style.fill = levelColors[0]
      el.style.cursor = 'pointer'

      el.addEventListener('click', onRegionClick)
  
    })
  }
})

function updateVisitLevel(newLevel) {
  if (selectedRegion.value) {
    visitData[selectedRegion.value].level = newLevel
    selectedLevel.value = newLevel
    // update SVG fill color here if needed, or trigger reactive update
    // e.g. updateRegionColor(selectedRegion.value, newLevel)
  }
}
</script>

<style scoped>
.popover {
  position: absolute;
  /* your styling */
  background: white;
  border: 1px solid #ccc;
  padding: 8px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.15);
  z-index: 9999;
}
svg {
  width: 100%;
  height: auto;
}
</style>

<!-- <template>
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
          {{ city.title }}: {{ city.count }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, reactive } from 'vue'

const svgContent = ref('')
const visitData = reactive<Record<string, { id: string; title: string; count: number }>>({})

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
        count: 0,
      }
    })
  }
})

function handleRegionClick(event: MouseEvent) {
  const target = event.target as HTMLElement
  const regionId = target?.id
  if (regionId && visitData[regionId] !== undefined) {
    visitData[regionId].count++

    // Change color directly (basic version — you can enhance with Vue bindings)
    target.setAttribute('fill', getColor(visitData[regionId].count))
  }
}

function getColor(count: number) {
  if (count >= 5) return '#f97316' // orange
  if (count >= 1) return '#60a5fa' // blue
  return '#e5e7eb' // gray
}

</script> -->