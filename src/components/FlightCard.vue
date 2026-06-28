<template>
    <section class="flight-section">
        <div class="section-header">
            <h2 class="section-title">Flight Options {{ city ? ` to ${city}` : '' }}</h2>
            <div class="sort-control">
                <label for="sort-select">Sort by</label>
                <select id="sort-select" v-model="sortBy">
                    <option value="price">Lowest Price</option>
                    <option value="duration">Shortest Duration</option>
                    <option value="departure">Earliest Departure</option>
                </select>
            </div>
        </div>

        <div v-if="error" class="error-text">{{ error }}</div>

        <div v-else class="flight-layout">
            <aside class="filters">
                <div class="filters-header">
                    <span>Filters</span>
                    <button class="clear-btn" @click="clearFilters" v-if="hasActiveFilters">Clear all</button>
                </div>

                <div class="filter-group">
                    <div class="filter-label">Stops</div>
                    <label class="checkbox-row">
                        <input type="checkbox" v-model="filters.nonStopOnly" />
                        Non-stop only
                    </label>
                </div>

                <div class="filter-group" v-if="airlines.length">
                    <div class="filter-label">Airlines</div>
                    <label v-for="airline in airlines" :key="airline.code" class="checkbox-row">
                        <input type="checkbox" v-model="filters.airlines" :value="airline.code" />
                        {{ airline.name }}
                        <span class="airline-count">({{ airline.count }})</span>
                    </label>
                </div>
            </aside>

            <div class="flight-list">
                <div class="results-count">Showing {{ visibleFlights.length }} of {{ flights.length }} flights</div>

                <div v-for="flight in displayedFlights" :key="flight.flightKey" class="flight-row"
                    :class="{ 'is-cheapest': flight.priceAmount === cheapestPrice }">
                    <div class="airline-info">
                        <img v-if="flight.airlineLogo" :src="flight.airlineLogo" :alt="flight.airlineName"
                            class="airline-logo" />
                        <div class="airline-name">{{ flight.airlineName }}</div>
                        <span v-if="flight.stops === 0" class="nonstop-badge">Non-stop</span>
                    </div>

                    <div class="route">
                        <div class="route-point">
                            <div class="route-date">{{ formatDate(flight.departureTime) }}</div>
                            <div class="route-time">{{ formatTime(flight.departureTime) }}</div>
                            <div class="route-code">{{ flight.from }}</div>
                        </div>
                        <div class="route-line">
                            <div class="duration-label">{{ flight.duration }}</div>
                            <div class="line"></div>
                        </div>
                        <div class="route-point">
                            <div class="route-time">{{ formatTime(flight.arrivalTime) }}</div>
                            <div class="route-code">{{ flight.to }}</div>
                        </div>
                    </div>

                    <div class="price-block">
                        <span v-if="flight.priceAmount === cheapestPrice" class="cheapest-tag">Cheapest</span>
                        <div class="price">{{ flight.priceFormatted }}</div>
                        <a :href="flight.bookingUrl" target="_blank" rel="noopener" class="book-btn">Select</a>
                    </div>
                </div>

                <button v-if="visibleCount < flights.length" class="show-more-btn" @click="visibleCount += 10">
                    Show more results
                </button>
            </div>
        </div>
    </section>
</template>

<script setup>
import { computed, reactive, ref } from 'vue'

const props = defineProps({
  flightsResult: { type: Object, required: true },
  city: { type: String, default: '' },
})

const error = computed(() => {
    if (props.flightsResult.status === 'error') return props.flightsResult.error
    if (props.flightsResult.status === 'ok' && !props.flightsResult.data?.data?.flightOffers) {
        return 'Flight results are temporarily unavailable. Please try searching again.'
    }
    return null
})

const cheapestPrice = computed(() => {
    if (!visibleFlights.value.length) return null
    return Math.min(...visibleFlights.value.map((f) => f.priceAmount))
})

const sortBy = ref('price')
const visibleCount = ref(10)
const filters = reactive({
    nonStopOnly: false,
    airlines: [],
})

const hasActiveFilters = computed(
    () => filters.nonStopOnly || filters.airlines.length > 0
)

function clearFilters() {
    filters.nonStopOnly = false
    filters.airlines = []
}

function formatDuration(totalSeconds) {
    const hours = Math.floor(totalSeconds / 3600)
    const minutes = Math.floor((totalSeconds % 3600) / 60)
    return `${hours}h ${minutes}m`
}

function formatMoney({ currencyCode, units, nanos }) {
    return { currencyCode, amount: units + nanos / 1e9 }
}

function formatTime(isoString) {
    return new Date(isoString).toLocaleTimeString('en-ZA', { hour: '2-digit', minute: '2-digit' })
}

function formatDate(isoString) {
    return new Date(isoString).toLocaleDateString('en-ZA', { weekday: 'short', day: 'numeric', month: 'short' })
}

const flights = computed(() => {
    if (props.flightsResult.status !== 'ok') return []
    const offers = props.flightsResult.data.data.flightOffers || []

    return offers.map((offer) => {
        const segment = offer.segments[0]
        const leg = segment.legs[0]
        const departDate = segment.departureTime.split('T')[0]
        const money = formatMoney(offer.priceBreakdown.total)
        const airlineCode = leg.carriersData?.[0]?.code ?? ''

        return {
            flightKey: offer.flightKey,
            from: segment.departureAirport.code,
            to: segment.arrivalAirport.code,
            departureTime: segment.departureTime,
            arrivalTime: segment.arrivalTime,
            durationSeconds: segment.totalTime,
            duration: formatDuration(segment.totalTime),
            stops: leg.flightStops?.length ?? 0,
            airlineCode,
            airlineName: leg.carriersData?.[0]?.name ?? '',
            airlineLogo: leg.carriersData?.[0]?.logo ?? null,
            priceAmount: money.amount,
            priceFormatted: `${money.currencyCode} ${money.amount.toFixed(2)}`,
            bookingUrl:
                `https://www.booking.com/flights/index.html?type=ONEWAY` +
                `&from=${segment.departureAirport.code}&to=${segment.arrivalAirport.code}&depart=${departDate}`,
        }
    })
})

// Airline checklist with counts, derived from the actual flights — not a
// separate API call, just a tally over data already in memory.
const airlines = computed(() => {
    const counts = new Map()
    for (const f of flights.value) {
        if (!f.airlineCode) continue
        const existing = counts.get(f.airlineCode)
        if (existing) {
            existing.count++
        } else {
            counts.set(f.airlineCode, { code: f.airlineCode, name: f.airlineName, count: 1 })
        }
    }
    return Array.from(counts.values()).sort((a, b) => b.count - a.count)
})

const visibleFlights = computed(() => {
    let result = flights.value

    if (filters.nonStopOnly) {
        result = result.filter((f) => f.stops === 0)
    }
    if (filters.airlines.length > 0) {
        result = result.filter((f) => filters.airlines.includes(f.airlineCode))
    }

    const sorted = [...result]
    if (sortBy.value === 'price') {
        sorted.sort((a, b) => a.priceAmount - b.priceAmount)
    } else if (sortBy.value === 'duration') {
        sorted.sort((a, b) => a.durationSeconds - b.durationSeconds)
    } else if (sortBy.value === 'departure') {
        sorted.sort((a, b) => new Date(a.departureTime) - new Date(b.departureTime))
    }

    return sorted
})

const displayedFlights = computed(() => visibleFlights.value.slice(0, visibleCount.value))
</script>

<style scoped>
.flight-section {
    margin-bottom: 48px;
}

.section-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 24px;
}

.section-title {
    font-family: var(--font-display);
    font-size: 24px;
    font-weight: 600;
    color: var(--color-primary);
}

.sort-control {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 13px;
    color: var(--color-on-surface-variant);
}

.sort-control select {
    font-family: var(--font-body);
    font-size: 13px;
    padding: 6px 10px;
    border: 1px solid var(--color-outline-variant);
    border-radius: var(--radius-md);
    background: white;
    cursor: pointer;
}

.flight-layout {
    display: grid;
    grid-template-columns: 220px 1fr;
    gap: 24px;
}

.filters {
    background: var(--color-surface-container-lowest);
    border: 1px solid var(--color-outline-variant);
    border-radius: var(--radius-lg);
    padding: 18px;
    align-self: start;
}

.filters-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-weight: 600;
    font-size: 14px;
    margin-bottom: 16px;
    color: var(--color-primary);
}

.clear-btn {
    background: none;
    border: none;
    color: var(--color-primary-container);
    font-size: 12px;
    cursor: pointer;
    text-decoration: underline;
}

.filter-group {
    margin-bottom: 18px;
}

.filter-label {
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 0.05em;
    text-transform: uppercase;
    color: var(--color-on-surface-variant);
    margin-bottom: 8px;
}

.checkbox-row {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 13px;
    padding: 4px 0;
    cursor: pointer;
}

.airline-count {
    color: var(--color-on-surface-variant);
    font-size: 11px;
}

.results-count {
    font-size: 13px;
    color: var(--color-on-surface-variant);
    margin-bottom: 14px;
}

.flight-list {
    display: flex;
    flex-direction: column;
    gap: 12px;
}

.flight-row {
    background: var(--color-surface-container-lowest);
    border: 1px solid var(--color-outline-variant);
    border-radius: var(--radius-lg);
    padding: 20px 24px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 24px;
    flex-wrap: wrap;
    transition: border-color 0.15s, transform 0.15s;
}

.flight-row:hover {
    border-color: var(--color-primary-container);
    transform: translateY(-1px);
}

.airline-info {
  display: flex;
  align-items: center;
  gap: 12px;
  min-width: 140px;
  max-width: 180px;
  flex-shrink: 0;
}
.airline-logo {
    width: 32px;
    height: 32px;
    object-fit: contain;
}

.airline-name {
    font-weight: 600;
    color: var(--color-primary);
}

.route {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 20px;
    min-width: 260px;
}

.route-point {
    text-align: center;
}

.route-time {
    font-weight: 700;
    color: var(--color-primary);
    font-size: 16px;
}

.route-code {
    font-size: 12px;
    color: var(--color-on-surface-variant);
}

.route-line {
    flex: 1;
    max-width: 140px;
    text-align: center;
}

.duration-label {
    font-size: 10px;
    font-weight: 700;
    letter-spacing: 0.05em;
    color: var(--color-on-surface-variant);
    margin-bottom: 4px;
}

.line {
    height: 2px;
    background: var(--color-outline-variant);
}

.price-block {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    gap: 8px;
    min-width: 110px;
}

.price {
    font-weight: 700;
    font-size: 18px;
    color: var(--color-primary-container);
}

.book-btn {
    background: var(--color-primary-container);
    color: white;
    text-decoration: none;
    padding: 6px 16px;
    border-radius: var(--radius-md);
    font-size: 13px;
    font-weight: 600;
    transition: filter 0.15s;
}

.book-btn:hover {
    filter: brightness(1.1);
}

.show-more-btn {
    align-self: center;
    margin-top: 8px;
    background: none;
    border: 1px solid var(--color-outline-variant);
    border-radius: var(--radius-md);
    padding: 10px 24px;
    font-family: var(--font-body);
    font-size: 13px;
    font-weight: 600;
    color: var(--color-primary-container);
    cursor: pointer;
    transition: border-color 0.15s;
}

.show-more-btn:hover {
    border-color: var(--color-primary-container);
}

.error-text {
    color: var(--color-error);
}

.route-date {
    font-size: 10px;
    font-weight: 600;
    color: var(--color-on-surface-variant);
    text-transform: uppercase;
    letter-spacing: 0.03em;
}

.flight-row.is-cheapest {
    border-color: var(--color-primary-container);
    border-width: 2px;
}

.nonstop-badge {
    display: inline-block;
    margin-top: 2px;
    font-size: 10px;
    font-weight: 700;
    color: var(--color-primary-container);
    background: rgba(0, 51, 102, 0.08);
    padding: 2px 6px;
    border-radius: 4px;
}

.cheapest-tag {
    font-size: 10px;
    font-weight: 700;
    color: white;
    background: var(--color-primary-container);
    padding: 2px 8px;
    border-radius: 999px;
    margin-bottom: 4px;
}
</style>