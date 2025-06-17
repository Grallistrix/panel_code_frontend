<script setup lang="ts">
const runtimeConfig = useRuntimeConfig()

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

// http://localhost:8000/sensor/{id}}/values?count={values_to_Read}

const { data } = useLazyAsyncData(
  'sensor-data',
  async () => {
    const reqs = AVAILABLE_SENSORS.map((v) =>
      $fetch(`${runtimeConfig.public.API_URL}/sensor/${v}/values?count=${currRecordsCount.value}`)
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
              return s
            })
          )

        return [v, scans]
      })
    )

    return sensorData
  },
  {
    watch: [currRecordsCount]
  }
)
</script>

<template>
  <div class="flex flex-col flex-1 index-page">
    <section class="bg-gray-50 dark:bg-gray-900">
      <div class="flex flex-col items-center justify-center px-6 py-8 mx-auto lg:py-0">
        <h1 class="text-xl font-bold leading-tight tracking-tight text-gray-900 md:text-2xl dark:text-white">
          Witaj w panleu IOT
        </h1>
        <div class="camera-stream">
          <h2>Podgląd kamery</h2>
          <img src="http://192.168.196.220:8080/" alt="Stream z kamery" />
        </div>
      </div>
    </section>

    <section>
      <TimeSelector @change="onTimeSelectorChange" />
    </section>

    <section>
      {{ data }}
    </section>
  </div>
</template>
