<template>
  <section class="attraction-section" v-if="shouldShow">
    <div class="section-header">
      <h2 class="section-title">Things to do{{ city ? ` in ${titleCity}` : '' }}</h2>
      <span class="section-meta" v-if="attractions.length">
        {{ attractions.length }} {{ attractions.length === 1 ? 'experience' : 'experiences' }}
      </span>
    </div>

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
          <div v-else class="attraction-image-fallback" aria-hidden="true">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"
              stroke-linecap="round" stroke-linejoin="round">
              <path d="M3 19l5.5-7 4 5 3-3.5L21 19H3z" />
              <circle cx="8" cy="7.5" r="2" />
            </svg>
          </div>
          <span v-if="attraction.freeCancellation" class="cancellation-badge">Free cancellation</span>
        </div>

        <div class="attraction-body">
          <span class="attraction-name">{{ attraction.name }}</span>
          <span class="attraction-price" v-if="attraction.price">
            <span class="price-from">From</span>
            {{ attraction.currency }} {{ Math.round(attraction.price).toLocaleString('en-ZA') }}
          </span>
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

const titleCity = computed(() =>
  props.city ? props.city.charAt(0).toUpperCase() + props.city.slice(1) : ''
)

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

.attraction-scroll {
  display: flex;
  gap: var(--space-4);
  overflow-x: auto;
  padding: var(--space-1) var(--space-1) var(--space-4);
  scroll-snap-type: x proximity;
}

.attraction-card {
  flex: 0 0 auto;
  width: 13rem;
  display: flex;
  flex-direction: column;
  text-decoration: none;
  color: inherit;
  background: var(--color-surface-container-lowest);
  border: 1px solid var(--color-outline-variant);
  border-radius: var(--radius-lg);
  overflow: hidden;
  box-shadow: var(--shadow-sm);
  scroll-snap-align: start;
  transition: transform var(--dur) var(--ease), box-shadow var(--dur) var(--ease),
    border-color var(--dur) var(--ease);
}

.attraction-card:hover {
  transform: translateY(-3px);
  box-shadow: var(--shadow-lg);
  border-color: var(--color-outline);
}

.attraction-image-wrap {
  position: relative;
  height: 8.5rem;
  background: var(--color-surface-container-low);
  overflow: hidden;
}

.attraction-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 400ms var(--ease);
}

.attraction-card:hover .attraction-image {
  transform: scale(1.04);
}

.attraction-image-fallback {
  width: 100%;
  height: 100%;
  display: grid;
  place-items: center;
  color: var(--color-outline);
}

.attraction-image-fallback svg {
  width: 30px;
  height: 30px;
}

.cancellation-badge {
  position: absolute;
  bottom: var(--space-2);
  left: var(--space-2);
  background: var(--color-surface-container-lowest);
  color: var(--teal-700);
  font-size: var(--text-2xs);
  font-weight: 700;
  letter-spacing: 0.02em;
  padding: var(--space-1) var(--space-2);
  border-radius: var(--radius-pill);
  box-shadow: var(--shadow-sm);
}

.attraction-body {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  flex: 1;
  padding: var(--space-3) var(--space-4) var(--space-4);
}

.attraction-name {
  font-size: var(--text-sm);
  font-weight: 600;
  color: var(--color-primary);
  line-height: 1.35;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.attraction-price {
  margin-top: auto;
  font-size: var(--text-sm);
  font-weight: 700;
  color: var(--color-primary);
  font-variant-numeric: tabular-nums;
}

.price-from {
  font-size: var(--text-xs);
  font-weight: 500;
  color: var(--color-on-surface-muted);
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
  background: var(--color-surface-container-low);
  border-radius: var(--radius-lg);
  color: var(--color-on-surface-variant);
  font-size: var(--text-sm);
  line-height: 1.6;
  max-width: 44rem;
}
</style>
