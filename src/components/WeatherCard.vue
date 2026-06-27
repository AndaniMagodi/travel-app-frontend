<template>
    <section class="weather-section">
      <h2 class="section-title">Weather Forecast</h2>
      <div v-if="error" class="error-text">{{ error }}</div>
      <div v-else class="forecast-grid">
        <div v-for="day in days" :key="day.date" class="day-card">
          <span class="day-label">{{ formatDate(day.date) }}</span>
          <div class="temps">
            <span class="temp-high">{{ Math.round(day.max) }}°</span>
            <span class="temp-low">{{ Math.round(day.min) }}°</span>
          </div>
          <div class="rain" v-if="day.rain > 0">💧 {{ day.rain }}mm</div>
        </div>
      </div>
    </section>
  </template>
  
  <script setup>
  import { computed } from 'vue'
  
  const props = defineProps({
    weatherResult: { type: Object, required: true },
  })
  
  const error = computed(() =>
    props.weatherResult.status === 'error' ? props.weatherResult.error : null
  )
  
  const days = computed(() => {
    if (props.weatherResult.status !== 'ok') return []
    const daily = props.weatherResult.data.daily
    return daily.time.map((date, i) => ({
      date,
      max: daily.temperature_2m_max[i],
      min: daily.temperature_2m_min[i],
      rain: daily.precipitation_sum[i],
    }))
  })
  
  function formatDate(dateStr) {
    const date = new Date(dateStr)
    return date.toLocaleDateString('en-ZA', { weekday: 'short', day: 'numeric' })
  }
  </script>
  
  <style scoped>
  .weather-section {
    margin-bottom: 48px;
  }
  
  .section-title {
    font-family: var(--font-display);
    font-size: 24px;
    font-weight: 600;
    color: var(--color-primary);
    margin-bottom: 24px;
  }
  
  .forecast-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(110px, 1fr));
    gap: 16px;
  }
  
  .day-card {
    background: var(--color-surface-container-lowest);
    border: 1px solid var(--color-outline-variant);
    border-radius: var(--radius-lg);
    padding: 16px;
    text-align: center;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
  }
  
  .day-label {
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 0.05em;
    text-transform: uppercase;
    color: var(--color-on-surface-variant);
  }
  
  .temps {
    margin: 12px 0;
    display: flex;
    justify-content: center;
    gap: 8px;
  }
  
  .temp-high {
    font-weight: 700;
    color: var(--color-primary);
  }
  
  .temp-low {
    color: var(--color-on-surface-variant);
  }
  
  .rain {
    font-size: 11px;
    font-weight: 600;
    color: var(--color-primary-container);
  }
  
  .error-text {
    color: var(--color-error);
  }
  </style>