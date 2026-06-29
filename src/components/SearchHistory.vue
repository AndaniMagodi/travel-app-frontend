<template>
    <div class="history-panel" v-if="entries.length > 0">
      <h3 class="history-title">Recent Searches</h3>
      <div class="history-list">
        <button
          v-for="entry in entries"
          :key="entry._id"
          class="history-item"
          @click="$emit('select', entry._id)"
        >
          <span class="entry-city">{{ entry.city }}</span>
          <span class="entry-dates">{{ entry.checkIn }} → {{ entry.checkOut }}</span>
        </button>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue'
  
  const entries = ref([])
  
  defineEmits(['select'])
  
  async function loadHistory() {
    try {
      const response = await fetch('http://localhost:4000/api/history')
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
  .history-panel {
    margin-bottom: 24px;
  }
  
  .history-title {
    font-size: 13px;
    font-weight: 600;
    letter-spacing: 0.05em;
    text-transform: uppercase;
    color: var(--color-on-surface-variant);
    margin-bottom: 10px;
  }
  
  .history-list {
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
  }
  
  .history-item {
    background: var(--color-surface-container-lowest);
    border: 1px solid var(--color-outline-variant);
    border-radius: 999px;
    padding: 8px 24px;
    font-family: var(--font-body);
    font-size: 13px;
    cursor: pointer;
    display: flex;
    gap: 8px;
    align-items: center;
    transition: border-color 0.15s;
  }
  
  .history-item:hover {
    border-color: var(--color-primary-container);
  }
  
  .entry-city {
    font-weight: 600;
    color: var(--color-primary);
    text-transform: capitalize;
  }
  
  .entry-dates {
    color: var(--color-on-surface-variant);
    font-size: 11px;
  }
  </style>