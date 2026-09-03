<template>
  <form class="search-form" @submit.prevent="handleSearch">
    <div class="field field-city">
      <label class="field-label" for="city-input">Destination city</label>
      <div class="input-wrap">
        <span class="input-icon" aria-hidden="true">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"
            stroke-linecap="round" stroke-linejoin="round">
            <path d="M20 10c0 6-8 12-8 12s-8-6-8-12a8 8 0 0 1 16 0z" />
            <circle cx="12" cy="10" r="3" />
          </svg>
        </span>
        <input id="city-input" v-model="city" type="text" placeholder="Cape Town" autocomplete="off" required />
      </div>
    </div>

    <div class="field">
      <label class="field-label" for="checkin-input">Check-in</label>
      <input id="checkin-input" v-model="checkIn" type="date" required />
    </div>

    <div class="field">
      <label class="field-label" for="checkout-input">Check-out</label>
      <input id="checkout-input" v-model="checkOut" type="date" required />
    </div>

    <button type="submit" class="search-btn" :disabled="loading">
      <span v-if="loading" class="spinner" aria-hidden="true"></span>
      <svg v-else class="btn-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
        stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
        <circle cx="11" cy="11" r="7" />
        <path d="m20 20-3.5-3.5" />
      </svg>
      {{ loading ? 'Searching' : 'Search' }}
    </button>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const city = ref('')
const checkIn = ref('')
const checkOut = ref('')
const loading = ref(false)

const emit = defineEmits(['results', 'loading'])

async function handleSearch() {
  loading.value = true
  emit('loading', true)
  try {
    const params = new URLSearchParams({ city: city.value, checkIn: checkIn.value, checkOut: checkOut.value })
    const response = await fetch(`${import.meta.env.VITE_API_URL}/api/trip?${params}`)
    const data = await response.json()
    emit('results', { ...data, _searchParams: { city: city.value, checkIn: checkIn.value, checkOut: checkOut.value } })
  } catch (err) {
    console.error('Search failed:', err)
  } finally {
    loading.value = false
    emit('loading', false)
  }
}
</script>

<style scoped>
/* No card chrome here — this form lives inside App.vue's .search-panel,
   which already provides the border, padding and elevation. */
.search-form {
  display: flex;
  align-items: flex-end;
  gap: var(--space-4);
  flex-wrap: wrap;
}

.field {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  min-width: 9.5rem;
}

.field-city {
  flex: 1;
  min-width: 14rem;
}

.field-label {
  font-size: var(--text-xs);
  font-weight: 600;
  letter-spacing: var(--tracking-caps);
  text-transform: uppercase;
  color: var(--color-on-surface-muted);
}

.input-wrap {
  position: relative;
}

.input-icon {
  position: absolute;
  left: var(--space-3);
  top: 50%;
  transform: translateY(-50%);
  color: var(--color-on-surface-muted);
  pointer-events: none;
  display: grid;
  place-items: center;
}

.input-icon svg {
  width: 17px;
  height: 17px;
}

input {
  width: 100%;
  font-size: var(--text-base);
  padding: 0.7rem var(--space-3);
  border: 1px solid var(--color-outline);
  border-radius: var(--radius-md);
  background: var(--color-surface-container-lowest);
  color: var(--color-on-surface);
  outline: none;
  transition: border-color var(--dur) var(--ease), box-shadow var(--dur) var(--ease);
}

.field-city input {
  padding-left: 2.4rem;
}

input::placeholder {
  color: var(--color-on-surface-muted);
}

input:hover {
  border-color: var(--color-on-surface-muted);
}

input:focus {
  border-color: var(--navy-700);
  box-shadow: 0 0 0 3px var(--navy-100);
}

.field-city:focus-within .input-icon {
  color: var(--navy-700);
}

.search-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-2);
  min-width: 8rem;
  background: var(--color-primary);
  color: white;
  border: none;
  padding: 0.75rem var(--space-5);
  border-radius: var(--radius-md);
  font-weight: 600;
  font-size: var(--text-base);
  cursor: pointer;
  box-shadow: var(--shadow-sm);
  transition: background var(--dur) var(--ease), transform var(--dur) var(--ease),
    box-shadow var(--dur) var(--ease);
}

.btn-icon {
  width: 16px;
  height: 16px;
}

.search-btn:hover:not(:disabled) {
  background: var(--navy-800);
  transform: translateY(-1px);
  box-shadow: var(--shadow-md);
}

.search-btn:active:not(:disabled) {
  transform: translateY(0);
}

.search-btn:disabled {
  opacity: 0.7;
  cursor: progress;
}

.spinner {
  width: 15px;
  height: 15px;
  border: 2px solid rgba(255, 255, 255, 0.35);
  border-top-color: white;
  border-radius: 50%;
  animation: spin 700ms linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

@media (max-width: 640px) {
  .search-form {
    gap: var(--space-3);
  }

  .field {
    flex: 1 1 100%;
  }

  .search-btn {
    width: 100%;
  }
}
</style>
