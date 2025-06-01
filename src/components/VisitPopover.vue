<!-- <template>
  <Listbox v-model="selectedLevel" :options="levels" optionLabel="label" class="w-full md:w-56" />
</template>

<script setup>
import { ref, defineProps } from 'vue';
import Listbox from 'primevue/listbox';

const props = defineProps({
  selected: Number,
})

const emit = defineEmits(['update:selected'])


const levelColors = [
  'bg-white border',
  'bg-blue-500',
  'bg-green-500',
  'bg-yellow-400',
  'bg-red-500',
  'bg-pink-400'
]

const levels = ref([
    { level: 0, label: '未踏' },
    { level: 1, label: '通過' },
    { level: 2, label: '接地' },
    { level: 3, label: '訪問' },
    { level: 4, label: '宿泊' },
    { level: 5, label: '居住' }
]);
</script> -->

<template>
  <Listbox
    v-model="innerSelected"
    :options="levels"
    optionLabel="label"
    class="w-full md:w-56"
  />
</template>

<script setup>
import { ref, watch, defineEmits, defineProps } from 'vue'
import Listbox from 'primevue/listbox'

const props = defineProps({
  selected: Object,
})

const emit = defineEmits(['update:selected'])

const levels = [
  { level: 0, label: '未踏' },
  { level: 1, label: '通過' },
  { level: 2, label: '接地' },
  { level: 3, label: '訪問' },
  { level: 4, label: '宿泊' },
  { level: 5, label: '居住' },
]

// internal state for the listbox model
const innerSelected = ref(props.selected)

// sync prop -> internal state
watch(
  () => props.selected,
  (newVal) => {
    innerSelected.value = newVal
  }
)

// emit change up to parent when user picks a new level
watch(innerSelected, (newVal) => {
  emit('update:selected', newVal)
})
</script>