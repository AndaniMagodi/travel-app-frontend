<template>
  <section class="car-section">
    <h2 class="section-title">Available Car Rentals</h2>
    <div v-if="error" class="error-text">{{ error }}</div>
    <div v-else class="car-grid">
      <div v-for="car in cars" :key="car.id" class="car-card">
        <span class="car-type">{{ car.type }}</span>
        <h3 class="car-name">{{ car.make }} {{ car.model }}</h3>
        <div class="car-footer">
          <div>
            <div class="car-price">{{ car.currency }} {{ car.pricePerDay }}</div>
            <div class="per-day">per day</div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  carRentalResult: { type: Object, required: true },
})

const error = computed(() =>
  props.carRentalResult.status === 'error' ? props.carRentalResult.error : null
)

const cars = computed(() => {
  if (props.carRentalResult.status !== 'ok') return []
  return props.carRentalResult.data.cars
})
</script>

<style scoped>
.car-section {
  margin-bottom: 48px;
}

.section-title {
  font-family: var(--font-display);
  font-size: 24px;
  font-weight: 600;
  color: var(--color-primary);
  margin-bottom: 24px;
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
  padding: 20px;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
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

.car-name {
  font-size: 18px;
  font-weight: 600;
  color: var(--color-primary);
  margin: 10px 0 16px;
}

.car-footer {
  border-top: 1px solid var(--color-outline-variant);
  padding-top: 12px;
}

.car-price {
  font-weight: 700;
  font-size: 18px;
  color: var(--color-primary-container);
}

.per-day {
  font-size: 11px;
  color: var(--color-on-surface-variant);
}

.error-text {
  color: var(--color-error);
}
</style>