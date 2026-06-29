<template>
  <section class="car-section">
    <div class="section-header">
      <h2 class="section-title">Available Car Rentals{{ city ? ` In ${city.charAt(0).toUpperCase() + city.slice(1)}` : '' }}</h2>
    </div>

    <div v-if="error" class="error-text">{{ error }}</div>
    <div v-else-if="cars.length === 0" class="empty-state">
      No car rentals found for these dates.
    </div>

    <div v-else class="car-layout">
      <aside class="filters">
        <div class="filters-header">
          <span>Filters</span>
          <button class="clear-btn" @click="selectedVendors = []" v-if="selectedVendors.length">Clear all</button>
        </div>

        <div class="filter-group" v-if="vendors.length">
          <div class="filter-label">Rental Company</div>
          <label v-for="vendor in vendors" :key="vendor.name" class="checkbox-row">
            <input type="checkbox" v-model="selectedVendors" :value="vendor.name" />
            {{ vendor.name }}
            <span class="vendor-count">({{ vendor.count }})</span>
          </label>
        </div>
      </aside>

      <div class="car-list">
        <div class="results-count">Showing {{ displayedCars.length }} of {{ filteredCars.length }} cars</div>

        <div class="car-grid">
          <div v-for="car in displayedCars" :key="car.id" class="car-card">
            <img v-if="car.image" :src="car.image" :alt="car.make" loading="lazy" class="car-image" />

            <div class="car-header">
              <span class="car-type">{{ car.type }}</span>
              <span v-if="car.freeCancellation" class="cancellation-badge">Free cancellation</span>
            </div>

            <h3 class="car-name">{{ car.make }}</h3>
            <div class="car-vendor">
              <img v-if="car.vendorLogo" :src="car.vendorLogo" :alt="car.vendor" class="vendor-logo" />
              {{ car.vendor }}
            </div>

            <div class="car-specs">
              <span class="spec" v-if="car.seats"><span class="spec-icon">👤</span>{{ car.seats }}</span>
              <span class="spec" v-if="car.transmission"><span class="spec-icon">⚙️</span>{{ car.transmission }}</span>
              <span class="spec" v-if="car.rating !== null"><span class="spec-icon">⭐</span>{{ car.rating }}%</span>
            </div>

            <div class="car-footer">
              <div v-if="car.pricePerDay !== null">
                <div class="car-price">{{ car.currency }} {{ Math.round(car.pricePerDay) }}</div>
                <div class="per-day">per day</div>
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
        </div>

        <button v-if="displayedCars.length < filteredCars.length" class="show-more-btn" @click="visibleCount += 8">
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
.car-section {
  margin-bottom: 32px;
}

.section-header {
  margin-bottom: 20px;
}

.section-title {
  font-family: var(--font-display);
  font-size: 24px;
  font-weight: 600;
  color: var(--color-primary);
}

.car-layout {
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

.vendor-count {
  color: var(--color-on-surface-variant);
  font-size: 11px;
}

.results-count {
  font-size: 13px;
  color: var(--color-on-surface-variant);
  margin-bottom: 14px;
}

.car-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
}

.car-card {
  background: var(--color-surface-container-lowest);
  border: 1px solid var(--color-outline-variant);
  border-radius: var(--radius-lg);
  padding: 16px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
  transition: border-color 0.15s, transform 0.15s;
}

.car-card:hover {
  border-color: var(--color-primary-container);
  transform: translateY(-2px);
}

.car-image {
  width: 100%;
  height: 100px;
  object-fit: contain;
  margin-bottom: 10px;
}

.car-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 8px;
  margin-bottom: 8px;
  flex-wrap: wrap;
}

.car-type {
  display: inline-block;
  background: rgba(0, 51, 102, 0.08);
  color: var(--color-primary-container);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  padding: 4px 8px;
  border-radius: 4px;
}

.cancellation-badge {
  font-size: 10px;
  font-weight: 600;
  color: #2e7d32;
}

.car-name {
  font-size: 15px;
  font-weight: 600;
  color: var(--color-primary);
  margin: 0 0 6px;
  line-height: 1.3;
}

.car-vendor {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 12px;
  color: var(--color-on-surface-variant);
  margin-bottom: 12px;
}

.vendor-logo {
  height: 14px;
  object-fit: contain;
}

.car-specs {
  display: flex;
  gap: 12px;
  margin-bottom: 14px;
  padding-bottom: 14px;
  border-bottom: 1px solid var(--color-outline-variant);
  flex-wrap: wrap;
}

.spec {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 12px;
  color: var(--color-on-surface-variant);
}

.car-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.car-price {
  font-weight: 700;
  font-size: 17px;
  color: var(--color-primary-container);
}

.per-day {
  font-size: 11px;
  color: var(--color-on-surface-variant);
}

.book-btn {
  background: var(--color-primary-container);
  color: white;
  text-decoration: none;
  padding: 8px 16px;
  border-radius: var(--radius-md);
  font-size: 13px;
  font-weight: 600;
  transition: filter 0.15s;
}

.book-btn:hover {
  filter: brightness(1.1);
}

.show-more-btn {
  display: block;
  margin: 16px auto 0;
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

.empty-state {
  padding: 20px 24px;
  background: var(--color-surface-container-low);
  border-radius: var(--radius-lg);
  color: var(--color-on-surface-variant);
  font-size: 14px;
}
</style>