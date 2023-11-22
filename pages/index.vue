<template>
  <section class="relative">
    <section id="tooltips">
      <!-- make a draggable for every tooltip -->
      <!-- https://vueuse.org/core/useDraggable/ -->
      <!-- <div
        v-for="(tooltip, index) in tooltips"
        :key="tooltip.id"
        class="z-50"
        @dblclick="removeTooltip(index)"
      >        
        <UseDraggable
          :initialValue="{ x: tooltip.x, y: tooltip.y }"
          class="absolute z-50 text-xs w-48"
          :storage-key="tooltip.id"
          storage-type="session"
          v-slot="{ x, y }"
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
      </div> -->

      <div
        v-for="(tooltip, index) in tooltips"
        :key="tooltip.id"
        class="z-50"
        ref="tooltipRefs"
        @dblclick="removeTooltip(index)"
      >
        <ScatterTooltip2
          :tooltip="tooltip"
          @positionChange="
            ($event) => tooltipPositionUpdate(index, $event.x, $event.y)
          "
        />
      </div>
    </section>

    <svg
      ref="overlay"
      class="w-screen h-screen absolute left-0 top-0 border border-red-300 z-20 pointer-events-none"
    >
      <!-- <line x1="0" y1="0" x2="100" y2="100" stroke="black" /> -->
      <line
        v-for="(tooltip, index) in tooltips"
        :key="tooltip.id"
        :x1="tooltip.x"
        :y1="tooltip.y"
        :x2="scatterPointMap.get(tooltip.id).x"
        :y2="scatterPointMap.get(tooltip.id).y"
        stroke="yellow"
        stroke-width="5"
      />
    </svg>

    <div class="p-10">
      <canvas ref="canvas" class="border z-10" />
    </div>
  </section>
</template>

<script setup>
import { set, get } from '@vueuse/core'
import * as d3 from 'd3'
import { UseDraggable } from '@vueuse/components'
import createScatterplot from 'regl-scatterplot'
import { onMounted } from 'vue'

const canvas = ref(null)
const { width, height } = useWindowSize()
const { x: mouseX, y: mouseY } = useMouse()

// const { width: canvasWidth, height: canvasHeight } = useElementSize(canvas)
const {
  x,
  y,
  top,
  right,
  width: canvasWidth,
  height: canvasHeight
} = useElementBounding(canvas)

let xScale = d3.scaleLinear()
let yScale = d3.scaleLinear()

const tooltips = ref([])

let scatterplot

function tooltipPositionUpdate(index, positionX, positionY) {
  console.log('New Position: ', index, positionX, positionY)
  // Handle the new position as needed

  // tooltips[index].x = positionX
  // tooltips[index].y = positionY

  tooltips.value[index].x = positionX
  tooltips.value[index].y = positionY
}

// const tooltipPointMap = new Map()
// GOAL: ?!
// const scatterPointMap = {
//   2309230: { x: 232, y: 343 }
// }
const scatterPointMap = ref(new Map())

onMounted(() => {
  const canvasEl = canvas.value
  scatterplot = createScatterplot({
    canvas: canvasEl,
    width: width.value,
    height: height.value,
    xScale,
    yScale,
    pointSize: 10
  })

  const color = '#FFFFFF'

  const points = new Array(10000)
    .fill()
    .map(() => [-1 + Math.random() * 2, -1 + Math.random() * 2, 0])

  // console.log(points)

  scatterplot.draw(points)
  // console.log('drawn')

  // subscribe to when points in the scatterplot are hovered
  scatterplot.subscribe('pointOver', (point) => {
    // console.log('pointOvered', point)
    handlePointOver(point)
  })

  // handle pointOut
  scatterplot.subscribe('select', (points) => {
    handleSelect(points)
  })

  // Listen for panning and zooming to update tooltip lines.
  scatterplot.subscribe('view', ({ xScale, yScale }) => {
    const newMap = new Map(
      scatterplot.get('pointsInView').map((pointIndex) => [
        pointIndex,
        {
          x: xScale(points[pointIndex][0]),
          y: yScale(points[pointIndex][1])
        }
      ])
    )
    // console.log(
    //   'pointsInScreenCoords',

    // )
    // make scatterPointMap

    // console.log(newMap)
    // scatterPointMap.value = newMap;
    set(scatterPointMap, newMap)
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
  // console.log('pointOvered', point)
}

function handlePointOut(point) {}

/* tooltip stuff */

function makeTooltip(point) {
  console.log('makeTooltip', point)

  const tooltip = reactive({
    id: point,
    name: 'tooltip',
    description: 'this is a tooltip',
    x: mouseX.value,
    y: mouseY.value
  })

  tooltips.value.push(tooltip)
}

function removeTooltip(index) {
  tooltips.value.splice(index, 1)
}
</script>

<style></style>
