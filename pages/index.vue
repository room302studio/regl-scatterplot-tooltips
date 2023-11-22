<template>
  <section class="relative">
    <section id="tooltips">
      <div
        v-for="(tooltip, index) in tooltips"
        :key="tooltip.id"
        class="z-50 relative"
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
      class="absolute left-0 top-0 border border-red-300 z-20 pointer-events-none"
      :width="width"
      :height="height"
    >
      <!-- <line x1="0" y1="0" x2="100" y2="100" stroke="black" /> -->
      <g v-for="(tooltip, index) in tooltips">
        <!-- <line
          v-if="scatterPointMap.has(tooltip.id)"
          :key="tooltip.id"
          :x1="tooltip.x"
          :y1="tooltip.y"
          :x2="scatterPointMap.get(tooltip.id).x"
          :y2="scatterPointMap.get(tooltip.id).y"
          stroke="yellow"
          stroke-width="5"
        /> -->

        <path
          v-if="scatterPointMap.has(tooltip.id)"
          :key="tooltip.id"
          :d="makeLineFor(tooltip, scatterPointMap.get(tooltip.id))"
          fill="transparent"
          stroke="yellow"
          stroke-width="5"
        />
      </g>
    </svg>

    <canvas ref="canvas" class="border z-10" />
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

// use d3 interpolate to make a :d for a path for a line between tooltip and scatter point
function makeLineFor(tooltip, scatterPoint) {
  const line = d3.line().curve(d3.curveCardinal)
  // // add an additional in-between point to curve the line
  // const tweenPoint = {
  //   x: (tooltip.x + scatterPoint.x) / 1.5,
  //   y: (tooltip.y + scatterPoint.y) / 2.2
  // }
  // Calculate the angle between tooltip and scatterPoint
  const angle =
    Math.atan2(scatterPoint.y - tooltip.y, scatterPoint.x - tooltip.x) +
    Math.PI / 2

  const distance = Math.sqrt(
    Math.pow(tooltip.x - scatterPoint.x, 2) +
      Math.pow(tooltip.y - scatterPoint.y, 2)
  )

  // Define the offset distance
  const offsetDistance = distance * 0.15 // Adjust this value as needed

  // Calculate the offset for tweenPoint based on the angle
  // const tweenPoint = {
  //   x: (tooltip.x + scatterPoint.x) / 1.95 + offsetDistance * Math.cos(angle),
  //   y: (tooltip.y + scatterPoint.y) / 1.95 + offsetDistance * Math.sin(angle)
  // }

  const weight = 0.25 // Adjust this value to move tweenPoint closer or further from tooltip
  const tweenPoint = {
    x:
      tooltip.x * (1 - weight) +
      scatterPoint.x * weight +
      offsetDistance * Math.cos(angle),
    y:
      tooltip.y * (1 - weight) +
      scatterPoint.y * weight +
      offsetDistance * Math.sin(angle)
  }

  const points = [
    [tooltip.x, tooltip.y],
    [tweenPoint.x, tweenPoint.y],
    [scatterPoint.x, scatterPoint.y]
  ]
  return line(points)
}

// const { width: canvasWidth, height: canvasHeight } = useElementSize(canvas)
const {
  x,
  y,
  top,
  right,
  width: canvasWidth,
  height: canvasHeight
} = useElementBounding(canvas)

watch([x], (oldX, newX) => {
  console.log('newX', newX)
})

let xScale = d3.scaleLinear().domain([-1, 1])
let yScale = d3.scaleLinear().domain([-1, 1])

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
