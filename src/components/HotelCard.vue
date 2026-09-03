<template>
  <section class="hotel-section">
    <div class="section-header">
      <h2 class="section-title">Top rated hotels{{ city ? ` in ${titleCity}` : '' }}</h2>
      <div class="controls">
        <label class="rating-filter" :class="{ 'is-on': minRatingOnly }">
          <input type="checkbox" v-model="minRatingOnly" />
          <span>8.0+ rated only</span>
        </label>
        <div class="sort-control">
          <label for="hotel-sort">Sort by</label>
          <select id="hotel-sort" v-model="sortBy">
            <option value="price">Lowest price</option>
            <option value="rating">Highest rated</option>
          </select>
        </div>
      </div>
    </div>

    <div v-if="error" class="error-text">{{ error }}</div>
    <template v-else>
      <div class="hotel-scroll">
        <article v-for="hotel in sortedHotels" :key="hotel.hotel_id" class="hotel-card">
          <div class="hotel-image-wrap">
            <img
              v-if="hotel.photoUrl"
              :src="hotel.photoUrl"
              :alt="hotel.name"
              class="hotel-image"
              loading="lazy"
            />
            <div v-else class="hotel-image-fallback" aria-hidden="true">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5"
                stroke-linecap="round" stroke-linejoin="round">
                <path d="M3 21V5.5A1.5 1.5 0 0 1 4.5 4h9A1.5 1.5 0 0 1 15 5.5V21" />
                <path d="M15 10h4.5A1.5 1.5 0 0 1 21 11.5V21M2 21h20" />
              </svg>
            </div>
            <div class="rating-badge" v-if="hotel.reviewCount > 0">
              <span class="rating-score">{{ hotel.reviewScore }}</span>
              <span class="review-count">{{ hotel.reviewCount }} reviews</span>
            </div>
          </div>

          <div class="hotel-body">
            <h3 class="hotel-name">{{ hotel.name }}</h3>
            <div class="hotel-footer">
              <div class="price-block" v-if="hotel.price">
                <span class="hotel-price">
                  <span class="price-currency">{{ hotel.priceCurrency }}</span>
                  {{ Math.round(hotel.price).toLocaleString('en-ZA') }}
                </span>
                <span class="per-night">per night</span>
              </div>
              <a :href="hotel.bookingUrl" target="_blank" rel="noopener" class="book-btn">Book</a>
            </div>
          </div>
        </article>
      </div>

      <div class="map-block" v-if="sortedHotels.length">
        <div class="map-header">
          <h3 class="map-title">Where these hotels are</h3>
          <span class="map-hint">Click a pin for the nightly rate</span>
        </div>
        <HotelMap :hotels="sortedHotels" />
      </div>
    </template>
  </section>
</template>

<script setup>
import { computed, ref } from "vue";
import HotelMap from './HotelMap.vue'

const props = defineProps({
  hotelsResult: { type: Object, required: true },
  usdToZarRate: { type: Number, default: null },
  city: { type: String, default: '' },
})

const sortBy = ref("price");
const minRatingOnly = ref(false);

const titleCity = computed(() =>
  props.city ? props.city.charAt(0).toUpperCase() + props.city.slice(1) : ''
)

// Checks the FULL path defensively — hotelsResult.data?.data?.hotels can
// be missing even when status === "ok" at the outer level, if the
// underlying provider's own response was itself a wrapped error (e.g. a
// transient auth hiccup from Booking.com). This must be checked before
// anything downstream (the hotel list, the map) tries to read .hotels.
const error = computed(() => {
  if (props.hotelsResult.status === "error") return props.hotelsResult.error;
  if (
    props.hotelsResult.status === "ok" &&
    !props.hotelsResult.data?.data?.hotels
  ) {
    return "Hotel results are temporarily unavailable. Please try searching again.";
  }
  return null;
});

// CONFIRMED: grossPrice is the TOTAL for the whole stay, not per-night —
// verified by comparing a 1-night vs 6-night search for the same hotels
// (the ratio matched stay length almost exactly). Divided by nights below
// to get the actual per-night rate the label promises.
//
// Booking.com returns hotel prices in USD regardless of the property's
// local currency. We convert to ZAR using a live exchange rate (passed down
// from App.vue / server.js) when available, falling back to USD display if
// the rate is unavailable for any reason.
//
// bookingUrl points to Booking.com's public search results page, pre-filled
// with this hotel's name and the searched dates — NOT a deep link to this
// exact offer (Booking.com's API doesn't expose one without a separate,
// more involved booking-flow integration). This continues the booking
// journey on Booking.com's real site rather than completing it on ours.
const hotels = computed(() => {
  if (props.hotelsResult.status !== "ok") return [];
  // Optional chaining all the way through — data.data can be a malformed
  // error-shaped object instead of undefined, so .hotels must be read
  // defensively even past the first ?. — this is the actual fix for the
  // crash: data.data.hotels (no ?. before .hotels) threw when data.data
  // existed but was shaped wrong.
  const rawHotels = props.hotelsResult.data?.data?.hotels || [];

  return rawHotels.map((h) => {
    const property = h.property;
    const priceInfo = property.priceBreakdown?.grossPrice;

    const checkinDate = new Date(property.checkinDate);
    const checkoutDate = new Date(property.checkoutDate);
    const nights = Math.max(
      1,
      Math.round((checkoutDate - checkinDate) / (1000 * 60 * 60 * 24))
    );

    let displayPrice = priceInfo?.value != null ? priceInfo.value / nights : null;
    let displayCurrency = priceInfo?.currency ?? "";

    if (displayCurrency === "USD" && props.usdToZarRate) {
      displayPrice = displayPrice * props.usdToZarRate;
      displayCurrency = "ZAR";
    }

    const bookingUrl =
      `https://www.booking.com/searchresults.html` +
      `?ss=${encodeURIComponent(property.name)}` +
      `&checkin=${property.checkinDate}` +
      `&checkout=${property.checkoutDate}`;

    return {
      hotel_id: h.hotel_id,
      name: property.name,
      reviewScore: property.reviewScore,
      reviewCount: property.reviewCount,
      photoUrl: property.photoUrls?.[0] || null,
      price: displayPrice,
      priceCurrency: displayCurrency,
      bookingUrl,
      latitude: property.latitude,
      longitude: property.longitude,
    };
  });
});

const sortedHotels = computed(() => {
  let result = hotels.value;

  if (minRatingOnly.value) {
    result = result.filter((h) => h.reviewScore >= 8.0);
  }

  const sorted = [...result];
  if (sortBy.value === "price") {
    sorted.sort((a, b) => (a.price ?? Infinity) - (b.price ?? Infinity));
  } else if (sortBy.value === "rating") {
    sorted.sort((a, b) => (b.reviewScore ?? 0) - (a.reviewScore ?? 0));
  }

  return sorted;
});
</script>

<style scoped>
.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
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

.controls {
  display: flex;
  align-items: center;
  gap: var(--space-4);
  flex-wrap: wrap;
}

.rating-filter {
  display: inline-flex;
  align-items: center;
  gap: var(--space-2);
  font-size: var(--text-sm);
  color: var(--color-on-surface-variant);
  cursor: pointer;
  padding: var(--space-2) var(--space-3);
  border: 1px solid var(--color-outline-variant);
  border-radius: var(--radius-pill);
  transition: border-color var(--dur) var(--ease), background var(--dur) var(--ease),
    color var(--dur) var(--ease);
}

.rating-filter:hover {
  border-color: var(--color-outline);
}

/* The chip itself reflects the filter state, so it's obvious at a glance
   whether the list is filtered. */
.rating-filter.is-on {
  background: var(--color-accent-soft);
  border-color: var(--teal-600);
  color: var(--teal-700);
  font-weight: 600;
}

.rating-filter input {
  accent-color: var(--teal-700);
  cursor: pointer;
  margin: 0;
}

.sort-control {
  display: flex;
  align-items: center;
  gap: var(--space-2);
  font-size: var(--text-sm);
  color: var(--color-on-surface-muted);
}

.sort-control select {
  font-size: var(--text-sm);
  padding: var(--space-2) var(--space-3);
  border: 1px solid var(--color-outline);
  border-radius: var(--radius-md);
  background: var(--color-surface-container-lowest);
  color: var(--color-on-surface);
  cursor: pointer;
  transition: border-color var(--dur) var(--ease);
}

.sort-control select:hover {
  border-color: var(--color-on-surface-muted);
}

/* ---- Card rail ---------------------------------------------------------- */
.hotel-scroll {
  display: flex;
  gap: var(--space-4);
  overflow-x: auto;
  padding: var(--space-1) var(--space-1) var(--space-4);
  scroll-snap-type: x proximity;
}

.hotel-card {
  flex: 0 0 auto;
  width: 16.5rem;
  display: flex;
  flex-direction: column;
  background: var(--color-surface-container-lowest);
  border: 1px solid var(--color-outline-variant);
  border-radius: var(--radius-lg);
  overflow: hidden;
  box-shadow: var(--shadow-sm);
  scroll-snap-align: start;
  transition: transform var(--dur) var(--ease), box-shadow var(--dur) var(--ease),
    border-color var(--dur) var(--ease);
}

.hotel-card:hover {
  transform: translateY(-3px);
  box-shadow: var(--shadow-lg);
  border-color: var(--color-outline);
}

.hotel-image-wrap {
  position: relative;
  height: 10.5rem;
  background: var(--color-surface-container-low);
  overflow: hidden;
}

.hotel-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 400ms var(--ease);
}

.hotel-card:hover .hotel-image {
  transform: scale(1.04);
}

.hotel-image-fallback {
  width: 100%;
  height: 100%;
  display: grid;
  place-items: center;
  color: var(--color-outline);
}

.hotel-image-fallback svg {
  width: 34px;
  height: 34px;
}

/* Gradient scrim so the rating stays readable over any photo. */
.hotel-image-wrap::after {
  content: '';
  position: absolute;
  inset: auto 0 0 0;
  height: 55%;
  background: linear-gradient(to top, rgba(0, 20, 43, 0.72), rgba(0, 20, 43, 0));
  pointer-events: none;
}

.rating-badge {
  position: absolute;
  bottom: var(--space-3);
  left: var(--space-3);
  z-index: 1;
  display: flex;
  align-items: baseline;
  gap: var(--space-2);
  color: white;
}

.rating-score {
  font-size: var(--text-md);
  font-weight: 700;
  font-variant-numeric: tabular-nums;
}

.review-count {
  font-size: var(--text-xs);
  color: rgba(255, 255, 255, 0.78);
}

.hotel-body {
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
  flex: 1;
  padding: var(--space-4);
}

.hotel-name {
  font-size: var(--text-md);
  font-weight: 600;
  color: var(--color-primary);
  line-height: 1.3;
  /* Two lines max, so cards in the rail stay the same height. */
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.hotel-footer {
  margin-top: auto;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  gap: var(--space-3);
}

.price-block {
  display: flex;
  flex-direction: column;
}

.hotel-price {
  font-size: var(--text-md);
  font-weight: 700;
  color: var(--color-primary);
  font-variant-numeric: tabular-nums;
}

.price-currency {
  font-size: var(--text-xs);
  font-weight: 600;
  color: var(--color-on-surface-muted);
}

.per-night {
  font-size: var(--text-xs);
  color: var(--color-on-surface-muted);
}

.book-btn {
  background: var(--color-primary);
  color: white;
  text-decoration: none;
  padding: var(--space-2) var(--space-4);
  border-radius: var(--radius-md);
  font-size: var(--text-sm);
  font-weight: 600;
  transition: background var(--dur) var(--ease);
}

.book-btn:hover {
  background: var(--navy-800);
}

/* ---- Map --------------------------------------------------------------- */
.map-block {
  margin-top: var(--space-4);
}

.map-header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: var(--space-2);
  margin-bottom: var(--space-3);
}

.map-title {
  font-size: var(--text-base);
  font-weight: 600;
  color: var(--color-primary);
}

.map-hint {
  font-size: var(--text-xs);
  color: var(--color-on-surface-muted);
}

.error-text {
  padding: var(--space-4);
  border-radius: var(--radius-md);
  background: var(--color-error-soft);
  color: var(--color-error);
  font-size: var(--text-sm);
}
</style>
