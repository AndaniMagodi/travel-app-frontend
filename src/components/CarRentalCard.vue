<template>
  <section class="car-section">
    <h2 class="section-title">Available Car Rentals{{ city ? ` in ${city}` : '' }}</h2>
    <div v-if="error" class="error-text">{{ error }}</div>
    <div v-else-if="cars.length === 0" class="empty-state">
      No car rentals found for these dates.
    </div>
    <div v-else class="car-grid">
      <div v-for="car in cars" :key="car.id" class="car-card">
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
  </section>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  carRentalResult: { type: Object, required: true },
  city: { type: String, default: '' },
})

const error = computed(() => {
  if (props.carRentalResult.status === 'error') return props.carRentalResult.error
  if (props.carRentalResult.status === 'ok' && !props.carRentalResult.data?.cars) {
    return 'Car rental results are temporarily unavailable. Please try searching again.'
  }
  return null
})

const cars = computed(() => {
  if (props.carRentalResult.status !== 'ok') return []
  return props.carRentalResult.data.cars || []
})
</script>

<style scoped>
.car-section {
  margin-bottom: 32px;
}

.section-title {
  font-family: var(--font-display);
  font-size: 24px;
  font-weight: 600;
  color: var(--color-primary);
  margin-bottom: 20px;
}

.car-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
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