<template>
    <section class="attraction-section" v-if="shouldShow">
      <h2 class="section-title">Things to Do{{ city ? ` In ${city.charAt(0).toUpperCase() + city.slice(1)}` : '' }}</h2>
  
      <div v-if="error" class="error-text">{{ error }}</div>
  
      <div v-else-if="attractions.length === 0" class="empty-state">
        No listed attractions found for {{ city || 'this destination' }} yet —
        this can happen for smaller towns. Try a nearby larger city to see this section in action.
      </div>
  
      <div v-else class="attraction-scroll">
        <a
          v-for="attraction in attractions"
          :key="attraction.id"
          :href="attraction.url"
          target="_blank"
          rel="noopener"
          class="attraction-card"
        >
          <div class="attraction-image-wrap">
            <img v-if="attraction.photoUrl" :src="attraction.photoUrl" :alt="attraction.name" loading="lazy" class="attraction-image" />
            <span v-if="attraction.freeCancellation" class="cancellation-badge">Free cancellation</span>
          </div>
          <div class="attraction-name">{{ attraction.name }}</div>
          <div class="attraction-price" v-if="attraction.price">
            From {{ attraction.currency }} {{ attraction.price.toFixed(0) }}
          </div>
        </a>
      </div>
    </section>
  </template>
  
  <script setup>
  import { computed } from 'vue'
  
  const props = defineProps({
    attractionsResult: { type: Object, required: true },
    city: { type: String, default: '' },
    checkIn: { type: String, default: '' },
    checkOut: { type: String, default: '' },
  })
  
  // Attractions is the one category where "no results" is a genuinely common,
  // expected outcome (smaller towns often have zero listings) rather than a
  // sign something broke — so this only shows an error banner for real
  // errors, and a calm explanatory message for the empty-but-ok case, never
  // hiding the section silently either way once a search has run.
  const shouldShow = computed(() => props.attractionsResult != null)
  
  const error = computed(() => {
    if (props.attractionsResult.status === 'error') return props.attractionsResult.error
    return null
  })
  
  const attractions = computed(() => {
    if (props.attractionsResult.status !== 'ok') return []
    const products = props.attractionsResult.data?.data?.products || []
  
    return products.slice(0, 10).map((p) => {
      const destId = p.ufiDetails?.ufi
      return {
        id: p.id,
        name: p.name,
        photoUrl: p.primaryPhoto?.small ?? p.primaryPhoto?.url ?? null,
        price: p.representativePrice?.chargeAmount ?? null,
        currency: p.representativePrice?.currency ?? '',
        freeCancellation: p.cancellationPolicy?.hasFreeCancellation ?? false,
        // Built from a real, confirmed-working Booking.com attractions search
        // URL pattern — dest_id is the location's ufi (already present per
        // product), dates come from the actual search. Affiliate-specific
        // params (aid, label) from the reference link were deliberately
        // dropped since those belong to whoever generated that link, not us.
        url: destId
          ? `https://www.booking.com/attractions/searchresults.en-gb.html?dest_id=${destId}&start_date=${props.checkIn}&end_date=${props.checkOut}`
          : `https://www.booking.com/attractions/searchresults.en-gb.html`,
      }
    })
  })
  </script>
  
  <style scoped>
  .attraction-section {
    margin-bottom: 32px;
  }
  
  .section-title {
    font-family: var(--font-display);
    font-size: 24px;
    font-weight: 600;
    color: var(--color-primary);
    margin-bottom: 20px;
  }
  
  .attraction-scroll {
    display: flex;
    gap: 24px;
    overflow-x: auto;
    padding-bottom: 8px;
  }
  
  .attraction-card {
    flex: 0 0 auto;
    width: 200px;
    text-decoration: none;
    color: inherit;
  }
  
  .attraction-image-wrap {
    position: relative;
    height: 130px;
    border-radius: var(--radius-lg);
    overflow: hidden;
    margin-bottom: 10px;
    background: var(--color-surface-container-low);
  }
  
  .attraction-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  
  .cancellation-badge {
    position: absolute;
    bottom: 8px;
    left: 8px;
    background: rgba(0, 0, 0, 0.55);
    color: white;
    font-size: 10px;
    font-weight: 600;
    padding: 3px 8px;
    border-radius: 999px;
  }
  
  .attraction-name {
    font-size: 14px;
    font-weight: 600;
    color: var(--color-primary);
    margin-bottom: 4px;
    line-height: 1.3;
  }
  
  .attraction-price {
    font-size: 13px;
    color: var(--color-primary-container);
    font-weight: 600;
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
    line-height: 1.5;
  }
  </style>