<template>
  <div
    ref="tooltipParent"
    :style="style"
    class="bg-white w-92 h-92 overflow-y-auto overflow-x-hidden p-2 rounded shadow select-none cursor-pointer absolute"
  >
    <div class="flex justify-between text-black">
      <div class="font-bold">{{ tooltip.name }} {{ x }} {{ y }}</div>
      <div>From data:{{ tooltip.x }} {{ tooltip.y }}</div>
      <div class="text-gray-500">{{ tooltip.id }}</div>
    </div>
    <div class="text-gray-500">{{ tooltip.description }}</div>
  </div>
</template>
<script setup>
const tooltipParent = ref(null)

// make draggable
const { x, y, style } = useDraggable(tooltipParent)

//watch x and y and console log when they change
watch([x, y], ([x, y]) => {
  console.log(x, y)
})

const { tooltip } = defineProps({
  tooltip: {
    type: Object,
    required: true
  }
})

// From the docs
// import { ref } from 'vue'
// import { useDraggable } from '@vueuse/core'

// const el = ref<HTMLElement | null>(null)

// // `style` will be a helper computed for `left: ?px; top: ?px;`
// const { x, y, style } = useDraggable(el, {
//   initialValue: { x: 40, y: 40 },
// })
</script>
