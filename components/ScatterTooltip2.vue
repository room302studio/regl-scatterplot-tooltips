<template>
  <div ref="tooltipParent" :style="style"
    class="bg-white w-92 h-92 overflow-y-auto overflow-x-hidden p-2 rounded shadow select-none cursor-pointer absolute">
    <div class="flex justify-between text-black">
      <div class="font-bold">{{ tooltip?.sample_name }}</div>
      <div class="text-gray-500">{{ tooltip?.id }}</div>
    </div>
    <div class="text-gray-500">{{ tooltip?.description }}</div>

    <slot />
  </div>
</template>
<script setup>
const { tooltip } = defineProps({
  tooltip: {
    type: Object,
    required: true
  }
})

const tooltipParent = ref(null)

// make draggable
const { x, y, style } = useDraggable(tooltipParent, {
  initialValue: {
    x: tooltip.x,
    y: tooltip.y
  }
})

// const { x, y, style } = useDraggable(el, {
//   initialValue: { x: 40, y: 40 },
// })

const emit = defineEmits(['positionChange'])

//watch x and y and console log when they change
watch([x, y], ([newX, newY]) => {
  console.log(newX, newY)
  emit('positionChange', { x: newX, y: newY })
})

// From the docs
// import { ref } from 'vue'
// import { useDraggable } from '@vueuse/core'

// const el = ref<HTMLElement | null>(null)

// // `style` will be a helper computed for `left: ?px; top: ?px;`
</script>
