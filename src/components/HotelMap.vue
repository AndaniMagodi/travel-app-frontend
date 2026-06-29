<template>
    <div class="map-wrap" v-if="hotels.length">
      <div ref="mapContainer" class="map-container"></div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, watch, nextTick } from 'vue'
  import L from 'leaflet'
  import 'leaflet/dist/leaflet.css'
  
  const props = defineProps({
    hotels: { type: Array, required: true }, // [{ name, latitude, longitude, price, priceCurrency }]
  })
  
  const mapContainer = ref(null)
  let mapInstance = null
  let markers = []
  
  function renderMap() {
    if (!mapContainer.value || !props.hotels.length) return
  
    if (!mapInstance) {
      mapInstance = L.map(mapContainer.value)
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors',
        maxZoom: 18,
      }).addTo(mapInstance)
    }
  
    // Clear old markers before re-rendering (e.g. on a new search)
    markers.forEach((m) => mapInstance.removeLayer(m))
    markers = []
  
    const bounds = []
    for (const hotel of props.hotels) {
      if (hotel.latitude == null || hotel.longitude == null) continue
      const marker = L.marker([hotel.latitude, hotel.longitude]).addTo(mapInstance)
      marker.bindPopup(`<strong>${hotel.name}</strong><br/>${hotel.priceCurrency} ${hotel.price?.toFixed(0) ?? '—'} / night`)
      markers.push(marker)
      bounds.push([hotel.latitude, hotel.longitude])
    }
  
    if (bounds.length) {
      mapInstance.fitBounds(bounds, { padding: [30, 30] })
    }
  }
  
  onMounted(async () => {
    await nextTick()
    renderMap()
  })
  
  watch(() => props.hotels, async () => {
    await nextTick()
    renderMap()
  })
  </script>
  
  <style scoped>
  .map-wrap {
    margin-top: 24px;
    border-radius: var(--radius-lg);
    overflow: hidden;
    border: 1px solid var(--color-outline-variant);
  }
  
  .map-container {
    height: 280px;
    width: 100%;
  }
  </style>