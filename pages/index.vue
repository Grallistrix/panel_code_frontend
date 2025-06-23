import 'leaflet/dist/leaflet.css'

<script setup lang="ts">
const runtimeConfig = useRuntimeConfig()
import ThreeViewer from '@/components/ThreeViewer.vue'

useHead({
  title: runtimeConfig.public.APP_TITLE
})

const currRecordsCount = ref(15)
const AVAILABLE_SENSORS = ['1', '2']

function onTimeSelectorChange({ option, recordsCount }: { option: string; recordsCount: number }) {
  console.info(option, recordsCount)
  currRecordsCount.value = recordsCount
  // await refreshNuxtData('sensor-data')
}

const chartKey = ref(0)

// http://localhost:8000/sensor/{id}}/values?count={values_to_Read}

const { data } = useLazyAsyncData(
  'sensor-data',
  async () => {
    const reqs = AVAILABLE_SENSORS.map((v) =>
      $fetch(`${runtimeConfig.public.API_URL}/new_sensor/${v}/values?count=${currRecordsCount.value}`)
    )

    const res = (await Promise.all(reqs)) as {
      sensor_id: string
      values: (Record<string, string> & { sensor_id?: string | number })[]
    }[]

    const sensorData = Object.fromEntries(
      AVAILABLE_SENSORS.map((v) => {
        const scans = res
          .filter((r) => r.sensor_id === v)
          .flatMap((r) =>
            r.values.map((s) => {
              delete s.sensor_id
              // eslint-disable-next-line max-nested-callbacks
              return Object.fromEntries(Object.entries(s).map(([k, val]) => [k, Number(val)]))
            })
          )

        return [v, scans]
      })
    )

    chartKey.value = ++chartKey.value

    return sensorData
  },
  {
    watch: [currRecordsCount]
  }
)

import { ref, watch, onMounted } from 'vue'

const firmy = ['ENEA', 'PGE', 'E.ON', 'TAURON', 'ENERGA']
const selectedFirma = ref(firmy[0])
const selectedDate = ref(new Date().toISOString().split('T')[0]) // YYYY-MM-DD
const selectedData = ref(null)

function formatDateToDDMMYYYY(isoDate) {
  const [year, month, day] = isoDate.split('-')
  return `${day}.${month}.${year}`
}

const fetchElectricityData = async () => {
  const dayParam = formatDateToDDMMYYYY(selectedDate.value)
  try {
    const response = await fetch(`http://localhost:8000/electricity/date?day=${dayParam}`)
    const json = await response.json()
    const found = json.results.find((item) => item.FIRMA === selectedFirma.value)
    selectedData.value = found || null
  } catch (error) {
    console.error('Błąd podczas pobierania danych:', error)
    selectedData.value = null
  }
}

watch([selectedFirma, selectedDate], fetchElectricityData, { immediate: true })

onMounted(fetchElectricityData)

import L from 'leaflet'

// Import ikon
import iconRetinaUrl from 'leaflet/dist/images/marker-icon-2x.png'
import iconUrl from 'leaflet/dist/images/marker-icon.png'
import shadowUrl from 'leaflet/dist/images/marker-shadow.png'
delete (L.Icon.Default.prototype as any)._getIconUrl
L.Icon.Default.mergeOptions({
  iconRetinaUrl,
  iconUrl,
  shadowUrl
})

const mapContainer = ref<HTMLDivElement | null>(null)
const map = ref<L.Map | null>(null)

// Współrzędne AGH Kraków
const latitude = 50.067127
const longitude = 19.916951
const zoomLevel = 15

onMounted(() => {
  if (!mapContainer.value) return

  map.value = L.map(mapContainer.value).setView([latitude, longitude], zoomLevel)

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
  }).addTo(map.value)

  L.marker([latitude, longitude])
    .addTo(map.value)
    .bindPopup('Wydział Inżynierii Metali i Informatyki Przemysłowej AGH')
    .openPopup()
})
</script>

<template>
  <div
    class="flex flex-col items-center justify-center flex-1 min-h-screen p-6 space-y-6 text-center text-white bg-blue-900 index-page"
  >
    <div
      class="flex flex-col items-center justify-center flex-1 min-h-screen p-6 space-y-6 text-center text-white bg-blue-900 index-page"
    >
      <!-- Sekcja powitalna -->
      <section class="w-full max-w-4xl p-6 text-gray-900 bg-white border border-blue-300 shadow-lg rounded-2xl">
        <h2 class="pb-2 mb-2 text-2xl font-bold border-b border-blue-300">Witaj użytkowniku!</h2>
        <h1 class="text-xl font-semibold">To twój panelu domu IoT (by Grallistrix Corp. INC)</h1>
      </section>

      <!-- KAMERA -->
      <section class="w-full max-w-4xl p-6 text-gray-900 bg-white border border-blue-300 shadow-lg rounded-2xl">
        <h2 class="pb-2 mb-4 text-2xl font-bold border-b border-blue-300">Podgląd kamery domofonu</h2>
        <img
          src="http://127.0.0.1:21370/"
          alt="Stream z kamery"
          class="mx-auto transition-transform duration-300 border border-gray-300 rounded-lg hover:scale-105"
        />
      </section>

      <!-- Wybór czasu -->
      <section class="w-full p-6 mx-auto text-gray-900 bg-white border border-blue-300 shadow-xl rounded-2xl">
        <h2 class="pb-2 mb-6 text-2xl font-bold text-center border-b border-blue-300">Dane z czujników</h2>

        <div class="flex justify-center mb-6">
          <TimeSelector @change="onTimeSelectorChange" />
        </div>

        <SensorChart v-if="data" :data="data" />
      </section>
    </div>

    <!-- PRĄD -->
    <section class="w-full max-w-5xl p-6 text-gray-900 bg-white border border-blue-300 shadow-lg rounded-2xl">
      <h2 class="pb-2 mb-4 text-2xl font-bold border-b border-blue-300">Cena Prądu</h2>
      <div class="flex flex-col justify-between gap-4 mb-6 md:flex-row md:items-center">
        <div class="grid grid-cols-2 gap-2 sm:grid-cols-3 md:grid-cols-5">
          <label v-for="firma in firmy" :key="firma" class="flex items-center gap-2">
            <input type="radio" :value="firma" v-model="selectedFirma" class="accent-blue-600" />
            {{ firma }}
          </label>
        </div>

        <div>
          <input
            type="date"
            v-model="selectedDate"
            @change="fetchElectricityData"
            class="px-3 py-1 border border-blue-300 rounded-md"
          />
        </div>
      </div>

      <!-- Tabela danych -->
      <div v-if="selectedData" class="grid grid-cols-3 gap-4 p-4 text-center bg-gray-100 rounded-xl">
        <div>
          <p class="text-sm font-semibold text-gray-500">TARYFA</p>
          <p class="text-lg font-bold">{{ selectedData.TARYFA }}</p>
        </div>
        <div>
          <p class="text-sm font-semibold text-gray-500">PRĄD</p>
          <p class="text-lg font-bold">{{ selectedData.PRAD }}</p>
        </div>
        <div>
          <p class="text-sm font-semibold text-gray-500">SUMA</p>
          <p class="text-lg font-bold">{{ selectedData.SUMA }}</p>
        </div>
      </div>

      <div v-else class="mt-4 text-sm italic text-gray-500">Brak danych do wyświetlenia.</div>
    </section>

    <!-- MAPA -->
    <section class="w-full max-w-5xl p-6 text-gray-900 bg-white border border-blue-300 shadow-lg rounded-2xl">
      <h2 class="pb-2 mb-4 text-2xl font-bold border-b border-blue-300">Lokalizacja Domu</h2>
      <div ref="mapContainer" class="w-full border border-gray-300 rounded-lg h-80" style="min-height: 320px"></div>
    </section>

    <!-- 3D VIEWER -->
    <section class="w-full max-w-5xl p-6 text-gray-900 bg-white border border-blue-300 shadow-lg rounded-2xl">
      <div
        class="flex flex-col items-center justify-center flex-1 min-h-screen p-6 space-y-6 text-center text-white bg-blue-900"
      >
        <ThreeViewer />
      </div>
    </section>
  </div>
</template>
