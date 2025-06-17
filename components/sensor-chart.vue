<script setup lang="ts">
import { Line } from 'vue-chartjs'

const $props = defineProps<{
  data: Record<string, Record<string, number>[]>
}>()

function getRandomColor() {
  const letters = '0123456789ABCDEF'
  let color = '#'
  for (let i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)]
  }
  return color
}

const labels = computed(() => {
  return [
    ...new Set(
      Object.entries($props.data).flatMap(([sensorId, values]) =>
        values.flatMap((v) => Object.keys(v).map((k) => `${sensorId} - ${k}`))
      )
    )
  ]
})

const chartData = computed(() => {
  if (Object.keys($props.data).length === 0) {
    return { labels: [], datasets: [] }
  }

  const datasets = labels.value.map((label) => {
    return {
      label,
      data: Object.entries($props.data).flatMap(([_, values]) =>
        values.flatMap((v, i) => {
          const value = v[label.split(' - ')[1]]
          return value !== undefined ? { x: i, y: value } : []
        })
      ),
      borderColor: getRandomColor(),
      backgroundColor: getRandomColor(),
      fill: false,
      tension: 0.2,
      yAxisID: label.includes('humidity') ? 'y1' : 'y'
    }
  })

  return {
    labels: Array.from({ length: $props.data[Object.keys($props.data)[0]].length }, (_, i) => `Record ${i + 1}`),
    datasets
  }
})

const CHART_OPTIONS = {
  responsive: false,
  maintainAspectRatio: true,
  interaction: {
    mode: 'index',
    intersect: false
  },
  stacked: false,
  scales: {
    y: {
      title: {
        display: true,
        text: 'Temperature'
      },
      type: 'linear',
      display: true,
      position: 'left'
    },
    y1: {
      title: {
        display: true,
        text: 'Humidity'
      },
      type: 'linear',
      display: true,
      position: 'right',
      grid: {
        drawOnChartArea: false
      }
    }
  },
  plugins: {
    legend: {
      display: true,
      position: 'top' as const
    }
  }
}
</script>

<template>
  <div class="sensor-chart">
    <div class="chart-header">
      <h2>Sensor Data Chart</h2>
    </div>
    <div class="chart-body">
      <div v-if="chartData.datasets.length === 0" class="no-data">No data available</div>
      <div v-else>
        <Line :data="chartData" style="height: 500px; width: calc(100vw - 3rem)" :options="CHART_OPTIONS" />
        {{ chartData }}
      </div>
    </div>
  </div>
</template>
