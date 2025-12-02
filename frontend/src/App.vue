<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import { getSensorInformation } from './api/netpie'
import type { SensorData } from './interface/sensor'

const loading = ref(false)
const error = ref('')

const sensorData = ref<SensorData>({
  temperature: 0,
  humid: 0,
  light: 0,
})

async function fetchData() {
  try {
    loading.value = true
    error.value = ''

    const fetchedSensorData = await getSensorInformation()

    if (!fetchedSensorData || !fetchedSensorData.data) {
      throw new Error('No data received from server')
    }

    sensorData.value = {
      temperature: Math.floor(fetchedSensorData.data.temperature ?? 0),
      humid: Math.floor(fetchedSensorData.data.humid ?? 0),
      light: Math.floor(fetchedSensorData.data.light ?? 0),
    }
  } catch (e: any) {
    console.error(e)
    error.value = e.message || 'Something went wrong while fetching data'
  } finally {
    loading.value = false
  }
}

let intervalId: number

onMounted(() => {
  fetchData()
  intervalId = setInterval(fetchData, 1000)
})

onUnmounted(() => {
  clearInterval(intervalId)
})
</script>

<template>
  <div
    class="min-h-screen bg-linear-to-br from-gray-100 to-gray-300 flex flex-col items-center p-6"
  >
    <h1 class="text-4xl font-bold mb-8 text-gray-900">Embed888 Dashboard</h1>

    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-8 w-full max-w-5xl">
      <div
        class="bg-white p-6 rounded-3xl shadow-xl transform transition hover:scale-105 hover:shadow-2xl"
      >
        <p class="text-gray-500 text-sm mb-2 uppercase tracking-wide font-semibold">Temperature</p>
        <p class="text-5xl font-bold text-blue-600">{{ sensorData.temperature }}°C</p>
      </div>

      <div
        class="bg-white p-6 rounded-3xl shadow-xl transform transition hover:scale-105 hover:shadow-2xl"
      >
        <p class="text-gray-500 text-sm mb-2 uppercase tracking-wide font-semibold">Humidity</p>
        <p class="text-5xl font-bold text-green-600">{{ sensorData.humid }}%</p>
      </div>

      <div
        class="bg-white p-6 rounded-3xl shadow-xl transform transition hover:scale-105 hover:shadow-2xl"
      >
        <p class="text-gray-500 text-sm mb-2 uppercase tracking-wide font-semibold">Light</p>
        <p class="text-5xl font-bold text-yellow-500">{{ sensorData.light }}</p>
      </div>
    </div>

    <div
      v-if="error"
      class="mt-6 p-4 bg-red-100 text-red-600 rounded-2xl max-w-5xl w-full text-center font-medium shadow-md animate-pulse"
    >
      {{ error }}
    </div>
  </div>
</template>
