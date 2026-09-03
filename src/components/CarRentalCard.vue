<template>
  <section class="car-section">
    <div class="section-header">
      <h2 class="section-title">Available car rentals{{ city ? ` in ${titleCity}` : '' }}</h2>
    </div>

    <div v-if="error" class="error-text">{{ error }}</div>
    <div v-else-if="cars.length === 0" class="empty-state">
      No car rentals found for these dates.
    </div>

    <div v-else class="car-layout">
      <aside class="filters">
        <div class="filters-header">
          <span>Filters</span>
          <button class="clear-btn" type="button" @click="selectedVendors = []" v-if="selectedVendors.length">
            Clear all
          </button>
        </div>

        <div class="filter-group" v-if="vendors.length">
          <div class="filter-label">Rental company</div>
          <label v-for="vendor in vendors" :key="vendor.name" class="checkbox-row">
            <input type="checkbox" v-model="selectedVendors" :value="vendor.name" />
            <span class="checkbox-text">{{ vendor.name }}</span>
            <span class="vendor-count">{{ vendor.count }}</span>
          </label>
        </div>
      </aside>

      <div class="car-list">
        <div class="results-count">
          Showing <strong>{{ displayedCars.length }}</strong> of {{ filteredCars.length }} cars
        </div>

        <div class="car-grid">
          <article v-for="car in displayedCars" :key="car.id" class="car-card">
            <div class="car-image-wrap">
              <img v-if="car.image" :src="car.image" :alt="car.make" loading="lazy" class="car-image" />
              <div v-else class="car-image-fallback" aria-hidden="true">
                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"
                  stroke-linecap="round" stroke-linejoin="round">
                  <path d="M4 16v-3l1.8-4.6A2 2 0 0 1 7.7 7h8.6a2 2 0 0 1 1.9 1.4L20 13v3M3 16h18" />
                  <circle cx="6.5" cy="16.5" r="1.6" />
                  <circle cx="17.5" cy="16.5" r="1.6" />
                </svg>
              </div>
            </div>

            <div class="car-body">
              <div class="car-header">
                <span class="car-type">{{ car.type }}</span>
                <span v-if="car.freeCancellation" class="cancellation-badge">Free cancellation</span>
              </div>

              <h3 class="car-name">{{ car.make }}</h3>

              <div class="car-vendor">
                <img v-if="car.vendorLogo" :src="car.vendorLogo" :alt="car.vendor" class="vendor-logo" />
                <span>{{ car.vendor }}</span>
              </div>

              <div class="car-specs">
                <span class="spec" v-if="car.seats">
                  <svg class="spec-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor"
                    stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                    <circle cx="12" cy="8" r="3.2" />
                    <path d="M5.5 20a6.5 6.5 0 0 1 13 0" />
                  </svg>
                  {{ car.seats }}
                </span>
                <span class="spec" v-if="car.transmission">
                  <svg class="spec-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor"
                    stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                    <path d="M7 4v16M7 8h5a2 2 0 0 0 2-2V4M7 14h5a2 2 0 0 1 2 2v4M17 4v6" />
                    <circle cx="7" cy="4" r="1.4" />
                    <circle cx="17" cy="4" r="1.4" />
                  </svg>
                  {{ car.transmission }}
                </span>
                <span class="spec" v-if="car.rating !== null">
                  <svg class="spec-icon" viewBox="0 0 24 24" fill="currentColor" aria-hidden="true">
                    <path d="m12 3.6 2.6 5.3 5.8.8-4.2 4.1 1 5.8-5.2-2.8-5.2 2.8 1-5.8L3.6 9.7l5.8-.8z" />
                  </svg>
                  {{ car.rating }}%
                </span>
              </div>

              <div class="car-footer">
                <div class="price-block" v-if="car.pricePerDay !== null">
                  <span class="car-price">
                    <span class="price-currency">{{ car.currency }}</span>
                    {{ Math.round(car.pricePerDay).toLocaleString('en-ZA') }}
                  </span>
                  <span class="per-day">per day</span>
                </div>
                <a
                  v-if="car.bookingUrl"
                  :href="car.bookingUrl"
                  target="_blank"
                  rel="noopener"
                  class="book-btn"
                  >Book</a
                >
              </div>
            </div>
          </article>
        </div>

        <button v-if="displayedCars.length < filteredCars.length" class="show-more-btn" type="button"
          @click="visibleCount += 8">
          Show more results
        </button>
      </div>
    </div>
  </section>
</template>

<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  carRentalResult: { type: Object, required: true },
  city: { type: String, default: '' },
})

const visibleCount = ref(8)
const selectedVendors = ref([])

const titleCity = computed(() =>
  props.city ? props.city.charAt(0).toUpperCase() + props.city.slice(1) : ''
)

const error = computed(() => {
  if (props.carRentalResult.status === 'error') return props.carRentalResult.error
  if (props.carRentalResult.status === 'ok' && !props.carRentalResult.data?.cars) {
    return 'Car rental results are temporarily unavailable. Please try searching again.'
  }
  return null
})

const cars = computed(() => {
  if (props.carRentalResult.status !== 'ok') return []
  return props.carRentalResult.data?.cars || []
})

// Vendor checklist with counts, derived from the actual cars returned —
// same pattern as the airline filter on FlightCard, no new API call needed.
const vendors = computed(() => {
  const counts = new Map()
  for (const c of cars.value) {
    if (!c.vendor) continue
    const existing = counts.get(c.vendor)
    if (existing) {
      existing.count++
    } else {
      counts.set(c.vendor, { name: c.vendor, count: 1 })
    }
  }
  return Array.from(counts.values()).sort((a, b) => b.count - a.count)
})

const filteredCars = computed(() => {
  if (selectedVendors.value.length === 0) return cars.value
  return cars.value.filter((c) => selectedVendors.value.includes(c.vendor))
})

const displayedCars = computed(() => filteredCars.value.slice(0, visibleCount.value))
</script>

<style scoped>
.section-header {
  margin-bottom: var(--space-5);
}

.section-title {
  font-family: var(--font-display);
  font-size: var(--text-xl);
  font-weight: 600;
  letter-spacing: -0.01em;
  color: var(--color-primary);
}

.car-layout {
  display: grid;
  grid-template-columns: 14rem 1fr;
  gap: var(--space-5);
  align-items: start;
}

/* ---- Filters (matches FlightCard's sidebar) ---------------------------- */
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

.vendor-count {
  margin-left: auto;
  font-size: var(--text-xs);
  font-variant-numeric: tabular-nums;
  color: var(--color-on-surface-muted);
  background: var(--color-surface-container-low);
  padding: 0 var(--space-2);
  border-radius: var(--radius-pill);
  flex-shrink: 0;
}

/* ---- Cars -------------------------------------------------------------- */
.results-count {
  font-size: var(--text-sm);
  color: var(--color-on-surface-muted);
  margin-bottom: var(--space-3);
}

.results-count strong {
  color: var(--color-on-surface-variant);
}

.car-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(13rem, 1fr));
  gap: var(--space-4);
}

.car-card {
  display: flex;
  flex-direction: column;
  background: var(--color-surface-container-lowest);
  border: 1px solid var(--color-outline-variant);
  border-radius: var(--radius-lg);
  overflow: hidden;
  box-shadow: var(--shadow-sm);
  transition: border-color var(--dur) var(--ease), box-shadow var(--dur) var(--ease),
    transform var(--dur) var(--ease);
}

.car-card:hover {
  border-color: var(--color-outline);
  box-shadow: var(--shadow-lg);
  transform: translateY(-3px);
}

/* A tinted plate behind the cut-out car photos — on plain white they float
   with no grounding. */
.car-image-wrap {
  height: 7.5rem;
  display: grid;
  place-items: center;
  padding: var(--space-3);
  background: linear-gradient(180deg, var(--color-surface-container-low), var(--color-surface-container-lowest));
  border-bottom: 1px solid var(--color-outline-variant);
}

.car-image {
  width: 100%;
  height: 100%;
  /* Grid items default to min-height:auto, which stops the image shrinking
     below its intrinsic height and makes it overflow the plate. */
  min-height: 0;
  object-fit: contain;
}

.car-image-fallback {
  color: var(--color-outline);
}

.car-image-fallback svg {
  width: 40px;
  height: 40px;
}

.car-body {
  display: flex;
  flex-direction: column;
  flex: 1;
  padding: var(--space-4);
}

.car-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: var(--space-2);
  margin-bottom: var(--space-3);
  flex-wrap: wrap;
}

.car-type {
  display: inline-block;
  background: var(--navy-100);
  color: var(--navy-800);
  font-size: var(--text-2xs);
  font-weight: 700;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  padding: 2px var(--space-2);
  border-radius: var(--radius-sm);
}

.cancellation-badge {
  font-size: var(--text-2xs);
  font-weight: 700;
  color: var(--teal-700);
}

.car-name {
  font-size: var(--text-base);
  font-weight: 600;
  color: var(--color-primary);
  line-height: 1.3;
  margin-bottom: var(--space-2);
}

.car-vendor {
  display: flex;
  align-items: center;
  gap: var(--space-2);
  font-size: var(--text-xs);
  color: var(--color-on-surface-muted);
  margin-bottom: var(--space-3);
}

.vendor-logo {
  height: 14px;
  object-fit: contain;
}

.car-specs {
  display: flex;
  gap: var(--space-3);
  flex-wrap: wrap;
  padding-bottom: var(--space-3);
  margin-bottom: var(--space-3);
  border-bottom: 1px solid var(--color-outline-variant);
}

.spec {
  display: inline-flex;
  align-items: center;
  gap: var(--space-1);
  font-size: var(--text-xs);
  color: var(--color-on-surface-variant);
}

.spec-icon {
  width: 13px;
  height: 13px;
  color: var(--color-on-surface-muted);
  flex-shrink: 0;
}

.car-footer {
  margin-top: auto;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  gap: var(--space-2);
}

.price-block {
  display: flex;
  flex-direction: column;
}

.car-price {
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

.per-day {
  font-size: var(--text-xs);
  color: var(--color-on-surface-muted);
}

.book-btn {
  background: var(--color-primary);
  color: white;
  text-decoration: none;
  padding: var(--space-2) var(--space-4);
  border-radius: var(--radius-md);
  font-size: var(--text-sm);
  font-weight: 600;
  transition: background var(--dur) var(--ease);
}

.book-btn:hover {
  background: var(--navy-800);
}

.show-more-btn {
  display: block;
  margin: var(--space-5) auto 0;
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

.empty-state {
  padding: var(--space-5);
  background: var(--color-surface-container-low);
  border-radius: var(--radius-lg);
  color: var(--color-on-surface-variant);
  font-size: var(--text-sm);
}

@media (max-width: 900px) {
  .car-layout {
    grid-template-columns: 1fr;
  }

  .filters {
    position: static;
  }
}
</style>
