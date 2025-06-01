<template>
  <div class="flex flex-col items-center px-4">
    <div class="my-4 p-4 bg-amber-300 rounded-2xl w-1/6 text-center">
      <h2 class="text-2xl font-bold">{{ totalScore }} 點</h2>
    </div>
    <div class="flex justify-between w-full max-w-[1024px] gap-6">
      <div class="w-1/3 ml-4 mt-4">
        <MapLegend />
      </div>
      <div class="flex-1">
        <div ref="svgContainer" class="h-[60vh]"></div>
      </div>
    </div>
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
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, onBeforeUnmount, computed } from 'vue'
import VisitPopover from './VisitPopover.vue'
import MapLegend from './MapLegend.vue'


const svgContainer = ref<HTMLElement | null>(null)
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
const levelColors = ['#FEF4E8', '#8FC9EC', '#92D7B3', '#FED264', '#FF9B75', '#F7B9C5']

const totalScore = computed(() => {
  return Object.values(visitData).reduce((sum, item) => sum + item.level.level, 0)
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
      const randomLevel = Math.floor(Math.random() * 6)
      visitData[id] = { title, level: levels[randomLevel] }

      el.style.fill = levelColors[randomLevel]
      el.style.cursor = 'pointer'
      el.classList.add('admin-zone')

      el.addEventListener('click', onRegionClick)
  
    })
  }

  // Adjust the size
  const svg = svgContainer.value.querySelector('svg')
  if (svg) {
    svg.removeAttribute('width')
    svg.removeAttribute('height')
    svg.setAttribute('width', '100%')
    svg.setAttribute('height', '100%')
    svg.setAttribute('preserveAspectRatio', 'xMidYMid meet')

    const padding_rate = 1.04
    
    // Optional: ensure it has viewBox to scale properly
    if (!svg.hasAttribute('viewBox')) {
      const bbox = svg.getBBox()
      svg.setAttribute('viewBox', `0 0 ${bbox.width} ${(bbox.height) * padding_rate}`)
    }
  }

  document.addEventListener('click', handleClickOutside)
})

onBeforeUnmount(() => {
  document.removeEventListener('click', handleClickOutside)
})

async function handleClickOutside(e) {
  const popoverEl = document.querySelector('.popover')
  const isAdminZone = e.target.closest('.admin-zone')
  if (
    popoverEl &&
    !popoverEl.contains(e.target) && // click is not inside popover
    !isAdminZone   // click is not on map
  ) {
    showPopover.value = false
  }
}

function updateVisitLevel(newLevel) {
  if (selectedRegion.value) {
    visitData[selectedRegion.value].level = newLevel
    selectedLevel.value = newLevel

    // update the SVG fill color
    const regionEl = svgContainer.value.querySelector(`#${selectedRegion.value}`)
    if (regionEl) {
      regionEl.style.fill = levelColors[newLevel.level]
    }
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