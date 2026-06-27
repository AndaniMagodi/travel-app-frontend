<template>
  <div class="app-shell">
    <header class="top-bar">
      <div class="brand">Travel Planner</div>
    </header>

    <div class="page">
      <div class="search-card">
        <SearchHistory ref="historyPanel" @select="loadPastSearch" />
        <TripSearch @results="handleResults" @loading="handleLoading" />
      </div>

      <SkeletonLoader v-if="loading" />

      <div v-else-if="tripData" class="results">
        <WeatherCard :weather-result="tripData.weather" />
        <HotelCard :hotels-result="tripData.hotels" />
        <FlightCard :flights-result="tripData.flights" />
        <CarRentalCard :car-rental-result="tripData.carRental" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import TripSearch from './components/TripSearch.vue'
import WeatherCard from './components/WeatherCard.vue'
import HotelCard from './components/HotelCard.vue'
import FlightCard from './components/FlightCard.vue'
import CarRentalCard from './components/CarRentalCard.vue'
import SearchHistory from './components/SearchHistory.vue'
import SkeletonLoader from './components/SkeletonLoader.vue'

const tripData = ref(null)
const loading = ref(false)
const historyPanel = ref(null)

function handleLoading(state) {
  loading.value = state
}

function handleResults(data) {
  tripData.value = data
  historyPanel.value?.loadHistory()
}

async function loadPastSearch(id) {
  loading.value = true
  const response = await fetch(`http://localhost:4000/api/history/${id}`)
  const entry = await response.json()
  tripData.value = entry.results
  loading.value = false
}
</script>

<style scoped>
.app-shell {
  min-height: 100vh;
}

.top-bar {
  background: var(--color-surface-container-lowest);
  border-bottom: 1px solid var(--color-outline-variant);
  padding: 18px 32px;
}

.brand {
  font-family: var(--font-display);
  font-size: 22px;
  font-weight: 600;
  color: var(--color-primary);
}

.page {
  max-width: 1100px;
  margin: 0 auto;
  padding: 40px 20px;
}

.search-card {
  background: var(--color-surface-container-lowest);
  border: 1px solid var(--color-outline-variant);
  border-radius: var(--radius-lg);
  padding: 24px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
  margin-bottom: 40px;
}
</style>