<template>
  <section class="weather-section">
    <div class="section-header">
      <h2 class="section-title">
        Weather forecast{{ city ? ` for ${titleCity}` : '' }}
      </h2>
      <span class="section-meta" v-if="days.length">{{ days.length }}-day outlook</span>
    </div>

    <div v-if="error" class="error-text">{{ error }}</div>
    <div v-else-if="days.length === 0" class="empty-state">
      No weather data available for these dates.
    </div>
    <div v-else class="weather-panel">
      <div class="day-list">
        <div v-for="day in days" :key="day.date" class="day-row">
          <span class="day-icon" :class="`is-${day.kind}`" aria-hidden="true">
            <!-- Sun -->
            <svg v-if="day.kind === 'sun'" viewBox="0 0 24 24" fill="none" stroke="currentColor"
              stroke-width="1.7" stroke-linecap="round">
              <circle cx="12" cy="12" r="4" />
              <path d="M12 2.5v2.2M12 19.3v2.2M2.5 12h2.2M19.3 12h2.2M5.3 5.3l1.6 1.6M17.1 17.1l1.6 1.6M18.7 5.3l-1.6 1.6M6.9 17.1l-1.6 1.6" />
            </svg>
            <!-- Part cloud -->
            <svg v-else-if="day.kind === 'cloud'" viewBox="0 0 24 24" fill="none" stroke="currentColor"
              stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round">
              <circle cx="8.5" cy="7.5" r="3" />
              <path d="M4 7.5H2.6M8.5 3V1.8M12 7.5h1.4M5.6 4.6 4.6 3.6M11.4 4.6l1-1" />
              <path d="M17.5 20H8a3.5 3.5 0 0 1 0-7 5 5 0 0 1 9.5 1.2 3 3 0 0 1 0 5.8z" />
            </svg>
            <!-- Showers -->
            <svg v-else-if="day.kind === 'showers'" viewBox="0 0 24 24" fill="none" stroke="currentColor"
              stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round">
              <path d="M17.5 15.5H8a3.5 3.5 0 0 1 0-7 5 5 0 0 1 9.5 1.2 3 3 0 0 1 0 5.8z" />
              <path d="M9 19v1.8M13 18.6v2.4M17 19v1.8" />
            </svg>
            <!-- Rain -->
            <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7"
              stroke-linecap="round" stroke-linejoin="round">
              <path d="M17.5 14.5H8a3.5 3.5 0 0 1 0-7 5 5 0 0 1 9.5 1.2 3 3 0 0 1 0 5.8z" />
              <path d="M8.5 17.5 7.5 21M12.5 17.5 11.5 21M16.5 17.5 15.5 21" />
            </svg>
          </span>

          <span class="day-label">
            <span class="day-weekday">{{ formatWeekday(day.date) }}</span>
            <span class="day-date">{{ formatDay(day.date) }}</span>
          </span>

          <span class="temp-low">{{ Math.round(day.min) }}°</span>
          <div class="temp-bar-track">
            <div class="temp-bar-fill" :style="{ left: day.barLeft + '%', width: day.barWidth + '%' }"></div>
          </div>
          <span class="temp-high">{{ Math.round(day.max) }}°</span>

          <span class="day-rain" :class="{ 'is-dry': day.rain === 0 }">
            {{ day.rain > 0 ? day.rain.toFixed(1) + ' mm' : '—' }}
          </span>
        </div>
      </div>

      <div class="stat-tiles" v-if="hasExtras">
        <div class="stat-tile">
          <span class="stat-label">Avg wind speed</span>
          <span class="stat-value">{{ avgWind }} <span class="stat-unit">km/h</span></span>
        </div>
        <div class="stat-tile">
          <span class="stat-label">Avg humidity</span>
          <span class="stat-value">{{ avgHumidity }}<span class="stat-unit">%</span></span>
        </div>
        <div class="stat-tile">
          <span class="stat-label">Warmest day</span>
          <span class="stat-value">{{ Math.round(warmest.max) }}<span class="stat-unit">°C</span></span>
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

const titleCity = computed(() =>
  props.city ? props.city.charAt(0).toUpperCase() + props.city.slice(1) : ''
)

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

// Returns a kind key rather than an emoji, so the template can pick a real
// vector icon and colour it with the app's palette.
function getWeatherKind(rain, maxTemp) {
  if (rain > 5) return 'rain'
  if (rain > 0) return 'showers'
  if (maxTemp >= 25) return 'sun'
  return 'cloud'
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
      kind: getWeatherKind(rain, max),
      barLeft: ((min - overallMin) / range) * 100,
      barWidth: ((max - min) / range) * 100,
    }
  })
})

const warmest = computed(() =>
  days.value.reduce((best, d) => (!best || d.max > best.max ? d : best), null)
)

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

function formatWeekday(dateStr) {
  return new Date(dateStr).toLocaleDateString('en-ZA', { weekday: 'short' })
}

function formatDay(dateStr) {
  return new Date(dateStr).toLocaleDateString('en-ZA', { day: 'numeric', month: 'short' })
}
</script>

<style scoped>
.section-header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
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

.section-meta {
  font-size: var(--text-xs);
  font-weight: 600;
  letter-spacing: var(--tracking-caps);
  text-transform: uppercase;
  color: var(--color-on-surface-muted);
}

.weather-panel {
  background: var(--color-surface-container-lowest);
  border: 1px solid var(--color-outline-variant);
  border-radius: var(--radius-lg);
  padding: var(--space-2) var(--space-5) var(--space-5);
  box-shadow: var(--shadow-sm);
}

.day-row {
  display: grid;
  grid-template-columns: 2rem 5.5rem 2.25rem 1fr 2.75rem 4rem;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3) 0;
  border-bottom: 1px solid var(--color-outline-variant);
}

.day-row:last-child {
  border-bottom: none;
}

.day-icon {
  display: grid;
  place-items: center;
}

.day-icon svg {
  width: 22px;
  height: 22px;
}

/* Sun stays warm, wet days go navy — the icon carries the information. */
.day-icon.is-sun {
  color: #c98a2b;
}

.day-icon.is-cloud {
  color: var(--navy-400);
}

.day-icon.is-showers,
.day-icon.is-rain {
  color: var(--navy-700);
}

.day-label {
  display: flex;
  flex-direction: column;
}

.day-weekday {
  font-size: var(--text-base);
  font-weight: 600;
  color: var(--color-primary);
}

.day-date {
  font-size: var(--text-xs);
  color: var(--color-on-surface-muted);
}

.temp-bar-track {
  position: relative;
  height: 6px;
  background: var(--color-surface-container);
  border-radius: var(--radius-pill);
}

.temp-bar-fill {
  position: absolute;
  top: 0;
  height: 6px;
  min-width: 6px;
  background: linear-gradient(90deg, var(--navy-600), #c98a2b);
  border-radius: var(--radius-pill);
  transition: left 300ms var(--ease), width 300ms var(--ease);
}

.temp-low,
.temp-high {
  font-variant-numeric: tabular-nums;
  text-align: right;
}

.temp-low {
  color: var(--color-on-surface-muted);
  font-size: var(--text-base);
}

.temp-high {
  font-weight: 700;
  color: var(--color-primary);
  font-size: var(--text-md);
}

.day-rain {
  font-size: var(--text-xs);
  font-variant-numeric: tabular-nums;
  text-align: right;
  color: var(--navy-700);
}

.day-rain.is-dry {
  color: var(--color-outline);
}

.stat-tiles {
  display: flex;
  gap: var(--space-5);
  flex-wrap: wrap;
  margin-top: var(--space-4);
  padding-top: var(--space-4);
  border-top: 1px solid var(--color-outline-variant);
}

.stat-tile {
  flex: 1;
  min-width: 7rem;
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
}

.stat-label {
  font-size: var(--text-xs);
  font-weight: 600;
  letter-spacing: var(--tracking-caps);
  text-transform: uppercase;
  color: var(--color-on-surface-muted);
}

.stat-value {
  font-size: var(--text-lg);
  font-weight: 700;
  color: var(--color-primary);
  font-variant-numeric: tabular-nums;
}

.stat-unit {
  font-size: var(--text-sm);
  font-weight: 500;
  color: var(--color-on-surface-variant);
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
  text-align: center;
  background: var(--color-surface-container-low);
  border-radius: var(--radius-lg);
  color: var(--color-on-surface-variant);
  font-size: var(--text-sm);
}

@media (max-width: 720px) {
  .day-row {
    grid-template-columns: 1.75rem 4.5rem 2rem 1fr 2.5rem;
    gap: var(--space-2);
  }

  .day-rain {
    display: none;
  }
}
</style>
