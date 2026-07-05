<template>
    <form class="search-bar" @submit.prevent="handleSearch">
      <div class="field city-field">
        <label class="field-label">Destination City</label>
        <div class="input-wrap">
          <span class="pin-icon">📍</span>
          <input v-model="city" type="text" placeholder="e.g. Cape Town" required />
        </div>
      </div>
  
      <div class="field">
        <label class="field-label">Check-in</label>
        <input v-model="checkIn" type="date" required />
      </div>
  
      <div class="field">
        <label class="field-label">Check-out</label>
        <input v-model="checkOut" type="date" required />
      </div>
  
      <button type="submit" class="search-btn" :disabled="loading">
        {{ loading ? "Searching..." : "Search" }}
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
  .search-bar {
    background: var(--color-surface-container-lowest);
    border: 1px solid var(--color-outline-variant);
    border-radius: var(--radius-lg);
    padding: 24px;
    display: flex;
    align-items: flex-end;
    gap: 24px;
    flex-wrap: wrap;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
    margin-bottom: 40px;
  }
  
  .field {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }
  
  .city-field {
    flex: 1;
    min-width: 220px;
  }
  
  .field-label {
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 0.05em;
    text-transform: uppercase;
    color: var(--color-on-surface-variant);
  }
  
  .input-wrap {
    position: relative;
  }
  
  .pin-icon {
    position: absolute;
    left: 12px;
    top: 50%;
    transform: translateY(-50%);
    font-size: 14px;
  }
  
  input {
    font-family: var(--font-body);
    font-size: 15px;
    padding: 12px 14px;
    border: 1px solid var(--color-outline-variant);
    border-radius: var(--radius-md);
    background: white;
    outline: none;
    transition: border-color 0.15s;
  }
  
  .city-field input {
    padding-left: 34px;
    width: 100%;
  }
  
  input:focus {
    border-color: var(--color-primary-container);
  }
  
  .search-btn {
    background: var(--color-primary-container);
    color: white;
    border: none;
    padding: 13px 28px;
    border-radius: var(--radius-md);
    font-family: var(--font-body);
    font-weight: 600;
    font-size: 15px;
    cursor: pointer;
    transition: filter 0.15s;
  }
  
  .search-btn:hover {
    filter: brightness(1.1);
  }
  
  .search-btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
  </style>