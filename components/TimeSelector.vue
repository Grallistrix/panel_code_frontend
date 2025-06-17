<template>
  <div class="w-full max-w-2xl p-6 bg-white rounded-lg shadow dark:border dark:bg-gray-800 dark:border-gray-700">
    <h2 class="mb-4 text-sm font-semibold text-center text-gray-900 dark:text-white">Wybierz zakres czasu</h2>

    <div class="flex justify-center gap-x-2">
      <button
        v-for="value in options"
        :key="value"
        :class="[
          'text-sm font-medium rounded-lg px-4 py-2 text-center border transition',
          selectedOption === value
            ? 'bg-primary-600 text-white dark:bg-primary-600 dark:text-white'
            : 'bg-gray-100 text-gray-900 dark:bg-gray-700 dark:text-white border-gray-300 dark:border-gray-600 hover:bg-gray-200 dark:hover:bg-gray-600'
        ]"
        @click="selectedOption = value"
      >
        {{ value }}
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import Qty from 'js-quantities/esm'

const options = ['15min', '1h', '1d', '7d', '31d']

const $emits = defineEmits<{
  (e: 'change', payload: { option: string; recordsCount: number }): void
}>()

const selectedOption = ref(options[0])
const currentRecordCount = computed(() => {
  if (!selectedOption.value) return 0

  return Qty(selectedOption.value).to('min').scalar
})

watch(selectedOption, (current) => $emits('change', { option: current, recordsCount: currentRecordCount.value }))
</script>
