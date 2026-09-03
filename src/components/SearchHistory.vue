<template>
  <div class="history-panel" v-if="entries.length > 0">
    <h3 class="history-title">Recent searches</h3>
    <div class="history-list">
      <button
        v-for="entry in entries"
        :key="entry._id"
        class="history-item"
        type="button"
        @click="$emit('select', entry._id)"
      >
        <span class="entry-icon" aria-hidden="true">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"
            stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="12" r="9" />
            <path d="M12 7.5V12l3 1.8" />
          </svg>
        </span>
        <span class="entry-city">{{ entry.city }}</span>
        <span class="entry-dates">{{ formatRange(entry.checkIn, entry.checkOut) }}</span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const entries = ref([])

defineEmits(['select'])

function formatRange(checkIn, checkOut) {
  const opts = { day: 'numeric', month: 'short' }
  const from = new Date(checkIn).toLocaleDateString('en-ZA', opts)
  const to = new Date(checkOut).toLocaleDateString('en-ZA', opts)
  return `${from} – ${to}`
}

async function loadHistory() {
  try {
    const response = await fetch(`${import.meta.env.VITE_API_URL}/api/history`)
    entries.value = await response.json()
  } catch (err) {
    console.error('Failed to load history:', err)
  }
}

// Expose so the parent can refresh the list after a new search completes
defineExpose({ loadHistory })

onMounted(loadHistory)
</script>

<style scoped>
/* Sits below the search form inside App.vue's .search-panel, separated by a
   rule rather than by another nested card. */
.history-panel {
  margin-top: var(--space-5);
  padding-top: var(--space-4);
  border-top: 1px solid var(--color-outline-variant);
}

.history-title {
  font-size: var(--text-xs);
  font-weight: 600;
  letter-spacing: var(--tracking-caps);
  text-transform: uppercase;
  color: var(--color-on-surface-muted);
  margin-bottom: var(--space-3);
}

.history-list {
  display: flex;
  gap: var(--space-2);
  flex-wrap: wrap;
}

.history-item {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  background: var(--color-surface-container-low);
  border: 1px solid transparent;
  border-radius: var(--radius-pill);
  padding: var(--space-2) var(--space-4);
  font-size: var(--text-sm);
  cursor: pointer;
  transition: background var(--dur) var(--ease), border-color var(--dur) var(--ease),
    transform var(--dur) var(--ease);
}

.history-item:hover {
  background: var(--navy-100);
  border-color: var(--navy-400);
  transform: translateY(-1px);
}

.entry-icon {
  display: grid;
  place-items: center;
  color: var(--color-on-surface-muted);
}

.entry-icon svg {
  width: 14px;
  height: 14px;
}

.history-item:hover .entry-icon {
  color: var(--navy-700);
}

.entry-city {
  font-weight: 600;
  color: var(--color-primary);
  text-transform: capitalize;
}

.entry-dates {
  color: var(--color-on-surface-variant);
  font-size: var(--text-xs);
}
</style>
