<template>
  <UseDraggable
    :initialValue="{ x: tooltip.x, y: tooltip.y }"
    class="absolute z-50 text-xs w-48"
    :storage-key="tooltip.id"
    storage-type="session"
  >
    <div class="bg-white p-2 rounded shadow select-none cursor-pointer">
      <div class="flex justify-between text-black">
        <div class="font-bold">{{ tooltip.name }} {{ x }} {{ y }}</div>
        <div>From data:{{ tooltip.x }} {{ tooltip.y }}</div>
        <div class="text-gray-500">{{ tooltip.id }}</div>
      </div>
      <div class="text-gray-500">{{ tooltip.description }}</div>
    </div>
  </UseDraggable>
</template>
<script setup>
import { UseDraggable } from '@vueuse/components'
const { tooltip } = defineProps({
  tooltip: {
    type: Object,
    required: true
  }
})
</script>
