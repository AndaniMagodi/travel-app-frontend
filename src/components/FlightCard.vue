<template>
    <section class="flight-section">
        <div class="section-header">
            <h2 class="section-title">Flight options{{ city ? ` to ${titleCity}` : '' }}</h2>
            <div class="sort-control">
                <label for="sort-select">Sort by</label>
                <select id="sort-select" v-model="sortBy">
                    <option value="price">Lowest price</option>
                    <option value="duration">Shortest duration</option>
                    <option value="departure">Earliest departure</option>
                </select>
            </div>
        </div>

        <div v-if="error" class="error-text">{{ error }}</div>

        <div v-else class="flight-layout">
            <aside class="filters">
                <div class="filters-header">
                    <span>Filters</span>
                    <button class="clear-btn" type="button" @click="clearFilters" v-if="hasActiveFilters">Clear all</button>
                </div>

                <div class="filter-group">
                    <div class="filter-label">Stops</div>
                    <label class="checkbox-row">
                        <input type="checkbox" v-model="filters.nonStopOnly" />
                        <span class="checkbox-text">Non-stop only</span>
                    </label>
                </div>

                <div class="filter-group" v-if="airlines.length">
                    <div class="filter-label">Airlines</div>
                    <label v-for="airline in airlines" :key="airline.code" class="checkbox-row">
                        <input type="checkbox" v-model="filters.airlines" :value="airline.code" />
                        <span class="checkbox-text">{{ airline.name }}</span>
                        <span class="airline-count">{{ airline.count }}</span>
                    </label>
                </div>
            </aside>

            <div class="flight-list">
                <div class="results-count">
                    Showing <strong>{{ visibleFlights.length }}</strong> of {{ flights.length }} flights
                </div>

                <article v-for="flight in displayedFlights" :key="flight.flightKey" class="flight-row"
                    :class="{ 'is-cheapest': flight.priceAmount === cheapestPrice }">
                    <div class="airline-info">
                        <img v-if="flight.airlineLogo" :src="flight.airlineLogo" :alt="flight.airlineName"
                            class="airline-logo" />
                        <div class="airline-text">
                            <div class="airline-name">{{ flight.airlineName }}</div>
                            <span v-if="flight.stops === 0" class="nonstop-badge">Non-stop</span>
                        </div>
                    </div>

                    <div class="route">
                        <div class="route-point">
                            <div class="route-date">{{ formatDate(flight.departureTime) }}</div>
                            <div class="route-time">{{ formatTime(flight.departureTime) }}</div>
                            <div class="route-code">{{ flight.from }}</div>
                        </div>

                        <div class="route-line">
                            <div class="duration-label">{{ flight.duration }}</div>
                            <div class="line" aria-hidden="true">
                                <span class="line-dot"></span>
                                <span class="line-plane">
                                    <svg viewBox="0 0 24 24" fill="currentColor">
                                        <path d="M21 15.5v-1.7l-7.5-4.3V4a1.5 1.5 0 0 0-3 0v5.5L3 13.8v1.7l7.5-2.2v4.3l-2.2 1.6v1.3l3.7-1 3.7 1v-1.3l-2.2-1.6v-4.3L21 15.5z"/>
                                    </svg>
                                </span>
                                <span class="line-dot"></span>
                            </div>
                        </div>

                        <div class="route-point">
                            <!-- Only shown when the flight actually lands on a different
                                 day — otherwise it just repeats the departure date. -->
                            <div class="route-date" :class="{ 'is-next-day': flight.landsNextDay }">
                                {{ flight.landsNextDay ? formatDate(flight.arrivalTime) : '' }}
                            </div>
                            <div class="route-time">{{ formatTime(flight.arrivalTime) }}</div>
                            <div class="route-code">{{ flight.to }}</div>
                        </div>
                    </div>

                    <div class="price-block">
                        <span v-if="flight.priceAmount === cheapestPrice" class="cheapest-tag">Cheapest</span>
                        <div class="price">
                            <span class="price-currency">{{ flight.priceCurrency }}</span>
                            {{ flight.priceValue }}
                        </div>
                        <a :href="flight.bookingUrl" target="_blank" rel="noopener" class="book-btn">Select</a>
                    </div>
                </article>

                <button v-if="visibleCount < flights.length" class="show-more-btn" type="button"
                    @click="visibleCount += 10">
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

const titleCity = computed(() =>
    props.city ? props.city.charAt(0).toUpperCase() + props.city.slice(1) : ''
)

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
            landsNextDay: segment.arrivalTime.split('T')[0] !== departDate,
            airlineCode,
            airlineName: leg.carriersData?.[0]?.name ?? '',
            airlineLogo: leg.carriersData?.[0]?.logo ?? null,
            priceAmount: money.amount,
            priceCurrency: money.currencyCode,
            // Whole rands with a thousands separator — the cents in these
            // fares are noise at this level of comparison.
            priceValue: Math.round(money.amount).toLocaleString('en-ZA'),
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
.section-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: var(--space-3);
    margin-bottom: var(--space-5);
}

.section-title {
    font-family: var(--font-display);
    font-size: var(--text-xl);
    font-weight: 600;
    letter-spacing: -0.01em;
    color: var(--color-primary);
}

.sort-control {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    font-size: var(--text-sm);
    color: var(--color-on-surface-muted);
}

.sort-control select {
    font-size: var(--text-sm);
    padding: var(--space-2) var(--space-3);
    border: 1px solid var(--color-outline);
    border-radius: var(--radius-md);
    background: var(--color-surface-container-lowest);
    color: var(--color-on-surface);
    cursor: pointer;
    transition: border-color var(--dur) var(--ease);
}

.sort-control select:hover {
    border-color: var(--color-on-surface-muted);
}

.flight-layout {
    display: grid;
    grid-template-columns: 14rem 1fr;
    gap: var(--space-5);
    align-items: start;
}

/* ---- Filters ----------------------------------------------------------- */
.filters {
    background: var(--color-surface-container-lowest);
    border: 1px solid var(--color-outline-variant);
    border-radius: var(--radius-lg);
    padding: var(--space-4);
    box-shadow: var(--shadow-sm);
    position: sticky;
    top: 5.5rem;
}

.filters-header {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
    font-weight: 600;
    font-size: var(--text-base);
    padding-bottom: var(--space-3);
    margin-bottom: var(--space-4);
    border-bottom: 1px solid var(--color-outline-variant);
    color: var(--color-primary);
}

.clear-btn {
    background: none;
    border: none;
    padding: 0;
    color: var(--navy-700);
    font-size: var(--text-xs);
    font-weight: 600;
    cursor: pointer;
    text-decoration: underline;
    text-underline-offset: 2px;
}

.clear-btn:hover {
    color: var(--navy-900);
}

.filter-group + .filter-group {
    margin-top: var(--space-5);
}

.filter-label {
    font-size: var(--text-xs);
    font-weight: 600;
    letter-spacing: var(--tracking-caps);
    text-transform: uppercase;
    color: var(--color-on-surface-muted);
    margin-bottom: var(--space-2);
}

.checkbox-row {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    font-size: var(--text-sm);
    padding: var(--space-1) 0;
    cursor: pointer;
    color: var(--color-on-surface-variant);
    transition: color var(--dur) var(--ease);
}

.checkbox-row:hover {
    color: var(--color-primary);
}

.checkbox-row input {
    accent-color: var(--navy-700);
    cursor: pointer;
    margin: 0;
    flex-shrink: 0;
}

.checkbox-text {
    min-width: 0;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.airline-count {
    margin-left: auto;
    font-size: var(--text-xs);
    font-variant-numeric: tabular-nums;
    color: var(--color-on-surface-muted);
    background: var(--color-surface-container-low);
    padding: 0 var(--space-2);
    border-radius: var(--radius-pill);
    flex-shrink: 0;
}

/* ---- Result rows ------------------------------------------------------- */
.results-count {
    font-size: var(--text-sm);
    color: var(--color-on-surface-muted);
    margin-bottom: var(--space-3);
}

.results-count strong {
    color: var(--color-on-surface-variant);
}

.flight-list {
    display: flex;
    flex-direction: column;
    gap: var(--space-3);
}

.flight-row {
    position: relative;
    background: var(--color-surface-container-lowest);
    border: 1px solid var(--color-outline-variant);
    border-radius: var(--radius-lg);
    padding: var(--space-4) var(--space-5);
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: var(--space-5);
    flex-wrap: wrap;
    box-shadow: var(--shadow-sm);
    transition: border-color var(--dur) var(--ease), box-shadow var(--dur) var(--ease),
        transform var(--dur) var(--ease);
}

.flight-row:hover {
    border-color: var(--color-outline);
    box-shadow: var(--shadow-md);
    transform: translateY(-2px);
}

/* The cheapest fare is marked with the accent and a left rail rather than a
   2px border, which used to shift the row's contents by a pixel. */
.flight-row.is-cheapest {
    border-color: var(--teal-600);
}

.flight-row.is-cheapest::before {
    content: '';
    position: absolute;
    inset: 0 auto 0 0;
    width: 3px;
    background: var(--teal-600);
    border-radius: var(--radius-lg) 0 0 var(--radius-lg);
}

.airline-info {
    display: flex;
    align-items: center;
    gap: var(--space-3);
    min-width: 9rem;
    max-width: 11rem;
    flex-shrink: 0;
}

.airline-logo {
    width: 34px;
    height: 34px;
    object-fit: contain;
    border-radius: var(--radius-sm);
    flex-shrink: 0;
}

.airline-text {
    min-width: 0;
}

.airline-name {
    font-weight: 600;
    font-size: var(--text-sm);
    color: var(--color-primary);
    line-height: 1.3;
}

.nonstop-badge {
    display: inline-block;
    margin-top: var(--space-1);
    font-size: var(--text-2xs);
    font-weight: 700;
    letter-spacing: 0.04em;
    text-transform: uppercase;
    color: var(--teal-700);
    background: var(--color-accent-soft);
    padding: 1px var(--space-2);
    border-radius: var(--radius-sm);
}

.route {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: var(--space-4);
    min-width: 17rem;
}

.route-point {
    text-align: center;
    min-width: 4.5rem;
}

.route-date {
    font-size: var(--text-2xs);
    font-weight: 600;
    letter-spacing: 0.04em;
    text-transform: uppercase;
    color: var(--color-on-surface-muted);
    /* Holds its line even when empty, so the departure and arrival times
       stay on the same baseline. */
    min-height: 1.1em;
}

.route-date.is-next-day {
    color: var(--teal-700);
}

.route-time {
    font-weight: 700;
    color: var(--color-primary);
    font-size: var(--text-lg);
    font-variant-numeric: tabular-nums;
    line-height: 1.25;
}

.route-code {
    font-size: var(--text-xs);
    font-weight: 600;
    letter-spacing: 0.06em;
    color: var(--color-on-surface-muted);
}

.route-line {
    flex: 1;
    max-width: 9rem;
    text-align: center;
}

.duration-label {
    font-size: var(--text-2xs);
    font-weight: 600;
    letter-spacing: 0.06em;
    text-transform: uppercase;
    color: var(--color-on-surface-muted);
    margin-bottom: var(--space-1);
}

.line {
    display: flex;
    align-items: center;
    gap: var(--space-1);
}

.line-dot {
    width: 5px;
    height: 5px;
    border-radius: 50%;
    background: var(--color-outline);
    flex-shrink: 0;
}

/* The plane sits on the connector so the row reads as a journey, not a
   dash between two numbers. */
.line-plane {
    flex: 1;
    display: flex;
    align-items: center;
    color: var(--navy-400);
}

.line-plane::before,
.line-plane::after {
    content: '';
    flex: 1;
    height: 1px;
    background: var(--color-outline-variant);
}

.line-plane svg {
    width: 13px;
    height: 13px;
    transform: rotate(90deg);
    margin: 0 2px;
}

.price-block {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    gap: var(--space-2);
    min-width: 7rem;
}

.price {
    font-weight: 700;
    font-size: var(--text-md);
    color: var(--color-primary);
    font-variant-numeric: tabular-nums;
}

.price-currency {
    font-size: var(--text-xs);
    font-weight: 600;
    color: var(--color-on-surface-muted);
}

.cheapest-tag {
    font-size: var(--text-2xs);
    font-weight: 700;
    letter-spacing: 0.04em;
    text-transform: uppercase;
    color: white;
    background: var(--teal-700);
    padding: 2px var(--space-2);
    border-radius: var(--radius-pill);
}

.book-btn {
    background: var(--color-primary);
    color: white;
    text-decoration: none;
    padding: var(--space-2) var(--space-5);
    border-radius: var(--radius-md);
    font-size: var(--text-sm);
    font-weight: 600;
    transition: background var(--dur) var(--ease);
}

.book-btn:hover {
    background: var(--navy-800);
}

.show-more-btn {
    align-self: center;
    margin-top: var(--space-2);
    background: var(--color-surface-container-lowest);
    border: 1px solid var(--color-outline);
    border-radius: var(--radius-pill);
    padding: var(--space-3) var(--space-6);
    font-size: var(--text-sm);
    font-weight: 600;
    color: var(--color-primary);
    cursor: pointer;
    transition: border-color var(--dur) var(--ease), box-shadow var(--dur) var(--ease);
}

.show-more-btn:hover {
    border-color: var(--navy-700);
    box-shadow: var(--shadow-sm);
}

.error-text {
    padding: var(--space-4);
    border-radius: var(--radius-md);
    background: var(--color-error-soft);
    color: var(--color-error);
    font-size: var(--text-sm);
}

@media (max-width: 900px) {
    .flight-layout {
        grid-template-columns: 1fr;
    }

    .filters {
        position: static;
    }
}

@media (max-width: 560px) {
    .flight-row {
        gap: var(--space-4);
    }

    .route {
        min-width: 100%;
        order: 3;
    }

    .price-block {
        align-items: flex-end;
    }
}
</style>
