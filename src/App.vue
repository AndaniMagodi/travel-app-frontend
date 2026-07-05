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

      <template v-else-if="tripData">
        <div class="results-header" v-if="activeSearch">
          Showing results for <strong>{{ activeSearch.city }}</strong>
          · {{ activeSearch.checkIn }} → {{ activeSearch.checkOut }}
        </div>

        <TripSummary :trip-data="tripData" />

        <div class="results-grid">
          <FlightCard :flights-result="tripData.flights" :city="activeSearch?.city" />
          <HotelCard :hotels-result="tripData.hotels" :usd-to-zar-rate="tripData.exchangeRate?.usdToZar"
            :city="activeSearch?.city" />
          <AttractionCard :attractions-result="tripData.attractions" :city="activeSearch?.city"
            :check-in="activeSearch?.checkIn" :check-out="activeSearch?.checkOut" />

          <div class="grid-compact">
            <WeatherCard :weather-result="tripData.weather" :city="activeSearch?.city" />
            <CarRentalCard :car-rental-result="tripData.carRental" :city="activeSearch?.city" />
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import TripSearch from './components/TripSearch.vue'
import WeatherCard from './components/WeatherCard.vue'
import HotelCard from './components/HotelCard.vue'
import FlightCard from './components/FlightCard.vue'
import CarRentalCard from './components/CarRentalCard.vue'
import SearchHistory from './components/SearchHistory.vue'
import SkeletonLoader from './components/SkeletonLoader.vue'
import TripSummary from './components/TripSummary.vue'
import AttractionCard from './components/AttractionCard.vue'

const tripData = ref(null)
const loading = ref(false)
const historyPanel = ref(null)

// Tracks which search is currently being displayed, so it's never ambiguous
// which city/dates the results on screen actually belong to.
const activeSearch = computed(() => tripData.value?._searchParams ?? null)

function handleLoading(state) {
  loading.value = state
}

function handleResults(data) {
  tripData.value = data
  historyPanel.value?.loadHistory()
}

async function loadPastSearch(id) {
  loading.value = true
  const response = await fetch(`${import.meta.env.VITE_API_URL}/api/history/${id}`)
  const entry = await response.json()
  tripData.value = {
    ...entry.results,
    _searchParams: { city: entry.city, checkIn: entry.checkIn, checkOut: entry.checkOut },
  }
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
  padding: 32px 20px;
}

.search-card {
  background: var(--color-surface-container-lowest);
  border: 1px solid var(--color-outline-variant);
  border-radius: var(--radius-lg);
  padding: 24px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
  margin-bottom: 32px;
}

.results-header {
  font-size: 14px;
  color: var(--color-on-surface-variant);
  margin-bottom: 24px;
  text-transform: capitalize;
}

.results-header strong {
  color: var(--color-primary);
  font-weight: 700;
}

.results-grid {
  display: flex;
  flex-direction: column;
  gap: 0;
}

.grid-compact {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
  align-items: start;
}

@media (max-width: 800px) {
  .grid-compact {
    grid-template-columns: 1fr;
  }
}
</style>