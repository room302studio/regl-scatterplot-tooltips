<template>
  <section class="relative">
    <section id="tooltips">
      <!-- make a draggable for every tooltip -->
      <div
        v-for="(tooltip, index) in tooltips"
        :key="tooltip.id"
        class="z-50"
        @dblclick="removeTooltip(index)"
      >
        <UseDraggable
          :initialValue="{ x: tooltip.x, y: tooltip.y }"
          class="absolute z-50 text-xs"
          :storage-key="tooltip.id"
          storage-type="session"
        >
          <div class="bg-white p-2 rounded shadow">
            <div class="flex justify-between">
              <div class="font-bold">{{ tooltip.name }}</div>
              <div class="text-gray-500">{{ tooltip.id }}</div>
            </div>
            <div class="text-gray-500">{{ tooltip.description }}</div>
          </div>
        </UseDraggable>
      </div>
    </section>

    <svg
      ref="overlay"
      class="w-screen h-screen absolute left-0 top-0 border border-red-300 z-20 pointer-events-none"
    ></svg>

    <canvas ref="canvas" class="border z-10" />
  </section>
</template>

<script setup>
import { UseDraggable } from '@vueuse/components'
import createScatterplot from 'regl-scatterplot'
import { onMounted } from 'vue'

const canvas = ref(null)
const { width, height } = useWindowSize()
const { x: mouseX, y: mouseY } = useMouse()

let scatterplot

onMounted(() => {
  const canvasEl = canvas.value
  scatterplot = createScatterplot({
    canvas: canvasEl,
    width: width.value,
    height: height.value,
    pointSize: 10
  })

  const color = '#FFFFFF'

  const points = new Array(10000)
    .fill()
    .map(() => [-1 + Math.random() * 2, -1 + Math.random() * 2, 0])

  console.log(points)

  scatterplot.draw(points)
  console.log('drawn')

  // subscribe to when points in the scatterplot are hovered
  scatterplot.subscribe('pointOver', (point) => {
    console.log('pointOvered', point)
    handlePointOver(point)
  })

  // handle pointOut
  scatterplot.subscribe('select', (points) => {
    handleSelect(points)
  })
})

function handleSelect({ points }) {
  console.log('selected', points)

  if (points.length === 1) {
    console.log('single point selected', points[0])
    makeTooltip(points[0])
  }
}

function handlePointOver(point) {
  console.log('pointOvered', point)
}

function handlePointOut(point) {}

/* tooltip stuff */
const tooltips = ref([])

function makeTooltip(point) {
  console.log('makeTooltip', point)
  const tooltip = {
    id: Math.random().toString(36).substring(7),
    name: 'tooltip',
    description: 'this is a tooltip',
    x: mouseX.value,
    y: mouseY.value
  }

  tooltips.value.push(tooltip)
}

function removeTooltip(index) {
  tooltips.value.splice(index, 1)
}
</script>

<style></style>
