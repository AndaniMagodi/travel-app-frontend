<template>
    <section class="weather-section">
      <h2 class="section-title">Weather Forecast{{ city ? ` in ${city}` : '' }}</h2>
      <div v-if="error" class="error-text">{{ error }}</div>
      <div v-else-if="days.length === 0" class="empty-state">
        No weather data available for these dates.
      </div>
      <div v-else class="weather-panel">
        <div class="day-list">
          <div v-for="day in days" :key="day.date" class="day-row">
            <span class="day-icon">{{ day.icon }}</span>
            <span class="day-label">{{ formatDate(day.date) }}</span>
            <div class="temp-bar-track">
              <div
                class="temp-bar-fill"
                :style="{ left: day.barLeft + '%', width: day.barWidth + '%' }"
              ></div>
            </div>
            <span class="temp-low">{{ Math.round(day.min) }}°</span>
            <span class="temp-high">{{ Math.round(day.max) }}°</span>
          </div>
        </div>
  
        <div class="stat-tiles" v-if="hasExtras">
          <div class="stat-tile">
            <span class="stat-label">Avg Wind Speed</span>
            <span class="stat-value">{{ avgWind }} km/h</span>
          </div>
          <div class="stat-tile">
            <span class="stat-label">Avg Humidity</span>
            <span class="stat-value">{{ avgHumidity }}%</span>
          </div>
        </div>
      </div>
    </section>
  </template>
  
  <script setup>
  import { computed } from 'vue'
  
  const props = defineProps({
    weatherResult: { type: Object, required: true },
    city: { type: String, default: '' },
  })
  
  const error = computed(() => {
    if (props.weatherResult.status === 'error') return props.weatherResult.error
    if (props.weatherResult.status === 'ok' && !props.weatherResult.data?.daily) {
      return 'Weather data is temporarily unavailable. Please try searching again.'
    }
    return null
  })
  
  const rawDaily = computed(() =>
    props.weatherResult.status === 'ok' ? props.weatherResult.data.daily : null
  )
  
  function getWeatherIcon(rain, maxTemp) {
    if (rain > 5) return '🌧️'
    if (rain > 0) return '🌦️'
    if (maxTemp >= 25) return '☀️'
    return '⛅'
  }
  
  const days = computed(() => {
    if (!rawDaily.value) return []
    const daily = rawDaily.value
    const allMins = daily.temperature_2m_min
    const allMaxs = daily.temperature_2m_max
    const overallMin = Math.min(...allMins)
    const overallMax = Math.max(...allMaxs)
    const range = overallMax - overallMin || 1
  
    return daily.time.map((date, i) => {
      const min = allMins[i]
      const max = allMaxs[i]
      const rain = daily.precipitation_sum[i]
      return {
        date,
        min,
        max,
        rain,
        icon: getWeatherIcon(rain, max),
        barLeft: ((min - overallMin) / range) * 100,
        barWidth: ((max - min) / range) * 100,
      }
    })
  })
  
  const hasExtras = computed(
    () => rawDaily.value?.wind_speed_10m_max && rawDaily.value?.relative_humidity_2m_mean
  )
  
  const avgWind = computed(() => {
    if (!hasExtras.value) return null
    const vals = rawDaily.value.wind_speed_10m_max
    return Math.round(vals.reduce((a, b) => a + b, 0) / vals.length)
  })
  
  const avgHumidity = computed(() => {
    if (!hasExtras.value) return null
    const vals = rawDaily.value.relative_humidity_2m_mean
    return Math.round(vals.reduce((a, b) => a + b, 0) / vals.length)
  })
  
  function formatDate(dateStr) {
    return new Date(dateStr).toLocaleDateString('en-ZA', { weekday: 'short', day: 'numeric' })
  }
  </script>
  
  <style scoped>
  .weather-section {
    margin-bottom: 32px;
  }
  
  .section-title {
    font-family: var(--font-display);
    font-size: 24px;
    font-weight: 600;
    color: var(--color-primary);
    margin-bottom: 20px;
  }
  
  .weather-panel {
    background: var(--color-surface-container-lowest);
    border: 1px solid var(--color-outline-variant);
    border-radius: var(--radius-lg);
    padding: 20px 24px;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04);
  }
  
  .day-row {
    display: grid;
    grid-template-columns: 28px 90px 1fr 32px 36px;
    align-items: center;
    gap: 12px;
    padding: 14px 0;
    border-bottom: 1px solid var(--color-outline-variant);
  }
  
  .day-row:last-child {
    border-bottom: none;
  }
  
  .day-icon {
    font-size: 22px;
  }
  
  .day-label {
    font-size: 14px;
    font-weight: 600;
    color: var(--color-primary);
  }
  
  .temp-bar-track {
    position: relative;
    height: 5px;
    background: var(--color-outline-variant);
    border-radius: 999px;
  }
  
  .temp-bar-fill {
    position: absolute;
    top: 0;
    height: 5px;
    background: var(--color-primary-container);
    border-radius: 999px;
    transition: left 0.3s, width 0.3s;
  }
  
  .temp-low {
    color: var(--color-on-surface-variant);
    font-size: 14px;
    text-align: right;
  }
  
  .temp-high {
    font-weight: 700;
    color: var(--color-primary);
    font-size: 15px;
    text-align: right;
  }
  
  .stat-tiles {
    display: flex;
    gap: 16px;
    margin-top: 18px;
    padding-top: 18px;
    border-top: 1px solid var(--color-outline-variant);
  }
  
  .stat-tile {
    flex: 1;
    display: flex;
    flex-direction: column;
    gap: 4px;
  }
  
  .stat-label {
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 0.05em;
    text-transform: uppercase;
    color: var(--color-on-surface-variant);
  }
  
  .stat-value {
    font-size: 18px;
    font-weight: 700;
    color: var(--color-primary);
  }
  
  .error-text {
    color: var(--color-error);
  }
  
  .empty-state {
    padding: 24px;
    text-align: center;
    color: var(--color-on-surface-variant);
    font-size: 14px;
  }
  </style>