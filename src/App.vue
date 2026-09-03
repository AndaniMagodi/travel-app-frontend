<template>
  <div class="app-shell">
    <header class="top-bar">
      <div class="top-bar-inner">
        <a class="brand" href="/">
          <span class="brand-mark" aria-hidden="true">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7"
              stroke-linecap="round" stroke-linejoin="round">
              <circle cx="12" cy="12" r="9" />
              <path d="M3.2 9h17.6M3.2 15h17.6" />
              <path d="M12 3a15 15 0 0 1 0 18a15 15 0 0 1 0-18z" />
            </svg>
          </span>
          <span class="brand-name">Travel Planner</span>
        </a>
        <span class="brand-tag">Weather · Flights · Stays · Cars</span>
      </div>
    </header>

    <main class="page">
      <!-- The hero only carries its weight before a search has run; once
           there are results on screen it would just push them down. -->
      <section class="hero" v-if="!tripData && !loading">
        <h1 class="hero-title">
          Everything about your trip,<br />
          <em>on one page.</em>
        </h1>
        <p class="hero-sub">
          Pick a destination and your dates. We'll pull the weather, flights,
          places to stay, cars and things to do into a single view.
        </p>
      </section>

      <!-- One panel, one border. The search form and the recent-search chips
           are sections inside it, not cards of their own. -->
      <section class="search-panel">
        <TripSearch @results="handleResults" @loading="handleLoading" />
        <SearchHistory ref="historyPanel" @select="loadPastSearch" />
      </section>

      <SkeletonLoader v-if="loading" />

      <template v-else-if="tripData">
        <div class="trip-context" v-if="activeSearch">
          <span class="context-city">{{ activeSearch.city }}</span>
          <span class="context-dates">
            {{ formatContextDate(activeSearch.checkIn) }}
            <span class="context-arrow" aria-hidden="true">→</span>
            {{ formatContextDate(activeSearch.checkOut) }}
          </span>
          <span class="context-nights">{{ nights }} {{ nights === 1 ? 'night' : 'nights' }}</span>
        </div>

        <TripSummary :trip-data="tripData" />

        <!-- Single column. These sections have wildly different heights
             (cars runs ~2700px, weather ~590px), so pairing any two of them
             in a two-column grid left one column mostly empty. -->
        <div class="results-stack">
          <FlightCard :flights-result="tripData.flights" :city="activeSearch?.city" />
          <HotelCard :hotels-result="tripData.hotels" :usd-to-zar-rate="tripData.exchangeRate?.usdToZar"
            :city="activeSearch?.city" />
          <WeatherCard :weather-result="tripData.weather" :city="activeSearch?.city" />
          <AttractionCard :attractions-result="tripData.attractions" :city="activeSearch?.city"
            :check-in="activeSearch?.checkIn" :check-out="activeSearch?.checkOut" />
          <CarRentalCard :car-rental-result="tripData.carRental" :city="activeSearch?.city" />
        </div>
      </template>
    </main>

    <footer class="site-footer">
      <span>Travel Planner</span>
      <span class="footer-note">
        Prices are indicative and provided by Booking.com, Expedia and Open-Meteo.
      </span>
    </footer>
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

const nights = computed(() => {
  const params = activeSearch.value
  if (!params?.checkIn || !params?.checkOut) return 1
  const checkin = new Date(params.checkIn)
  const checkout = new Date(params.checkOut)
  return Math.max(1, Math.round((checkout - checkin) / (1000 * 60 * 60 * 24)))
})

function formatContextDate(dateStr) {
  if (!dateStr) return ''
  return new Date(dateStr).toLocaleDateString('en-ZA', {
    day: 'numeric',
    month: 'short',
    year: 'numeric',
  })
}

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
  display: flex;
  flex-direction: column;
}

/* ---- Top bar ------------------------------------------------------------ */
.top-bar {
  position: sticky;
  top: 0;
  z-index: 50;
  background: rgba(255, 255, 255, 0.82);
  backdrop-filter: saturate(1.6) blur(10px);
  border-bottom: 1px solid var(--color-outline-variant);
}

.top-bar-inner {
  max-width: var(--page-max);
  margin: 0 auto;
  padding: var(--space-4) var(--space-5);
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-4);
}

.brand {
  display: inline-flex;
  align-items: center;
  gap: var(--space-3);
  text-decoration: none;
}

.brand-mark {
  display: grid;
  place-items: center;
  width: 30px;
  height: 30px;
  border-radius: var(--radius-sm);
  background: var(--color-primary);
  color: white;
}

.brand-mark svg {
  width: 18px;
  height: 18px;
}

.brand-name {
  font-family: var(--font-display);
  font-size: var(--text-lg);
  font-weight: 700;
  letter-spacing: -0.01em;
  color: var(--color-primary);
}

.brand-tag {
  font-size: var(--text-xs);
  font-weight: 500;
  letter-spacing: var(--tracking-caps);
  text-transform: uppercase;
  color: var(--color-on-surface-muted);
}

/* ---- Page -------------------------------------------------------------- */
.page {
  flex: 1;
  width: 100%;
  max-width: var(--page-max);
  margin: 0 auto;
  padding: var(--space-7) var(--space-5) var(--space-8);
}

/* ---- Hero -------------------------------------------------------------- */
.hero {
  max-width: 44rem;
  margin-bottom: var(--space-6);
}

.hero-title {
  font-family: var(--font-display);
  font-size: var(--text-hero);
  font-weight: 700;
  letter-spacing: -0.02em;
  color: var(--color-primary);
  margin-bottom: var(--space-4);
}

.hero-title em {
  font-style: italic;
  color: var(--navy-600);
}

.hero-sub {
  max-width: 34rem;
  font-size: var(--text-md);
  line-height: 1.6;
  color: var(--color-on-surface-variant);
}

/* ---- Search panel ------------------------------------------------------ */
.search-panel {
  background: var(--color-surface-container-lowest);
  border: 1px solid var(--color-outline-variant);
  border-radius: var(--radius-xl);
  padding: var(--space-5);
  box-shadow: var(--shadow-md);
  margin-bottom: var(--space-7);
}

/* ---- Trip context ------------------------------------------------------ */
.trip-context {
  display: flex;
  align-items: baseline;
  flex-wrap: wrap;
  gap: var(--space-3) var(--space-4);
  padding-bottom: var(--space-4);
  margin-bottom: var(--space-6);
  border-bottom: 1px solid var(--color-outline-variant);
}

.context-city {
  font-family: var(--font-display);
  font-size: var(--text-2xl);
  font-weight: 700;
  letter-spacing: -0.02em;
  color: var(--color-primary);
  text-transform: capitalize;
}

.context-dates {
  font-size: var(--text-base);
  color: var(--color-on-surface-variant);
}

.context-arrow {
  color: var(--color-on-surface-muted);
  padding: 0 var(--space-1);
}

.context-nights {
  font-size: var(--text-xs);
  font-weight: 600;
  letter-spacing: var(--tracking-caps);
  text-transform: uppercase;
  color: var(--color-accent);
  background: var(--color-accent-soft);
  padding: var(--space-1) var(--space-3);
  border-radius: var(--radius-pill);
}

/* ---- Results ----------------------------------------------------------- */
.results-stack {
  display: flex;
  flex-direction: column;
  gap: var(--space-8);
}

/* ---- Footer ------------------------------------------------------------ */
.site-footer {
  border-top: 1px solid var(--color-outline-variant);
  padding: var(--space-5);
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-3);
  max-width: var(--page-max);
  margin: 0 auto;
  width: 100%;
  font-size: var(--text-sm);
  color: var(--color-on-surface-muted);
}

.site-footer > span:first-child {
  font-family: var(--font-display);
  font-weight: 600;
  color: var(--color-on-surface-variant);
}

.footer-note {
  max-width: 32rem;
}

@media (max-width: 640px) {
  .page {
    padding: var(--space-5) var(--space-4) var(--space-7);
  }

  .search-panel {
    padding: var(--space-4);
  }

  .brand-tag {
    display: none;
  }

  .results-stack {
    gap: var(--space-7);
  }
}
</style>
