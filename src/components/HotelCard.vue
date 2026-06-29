<template>
    <section class="hotel-section">
      <div class="section-header">
        <h2 class="section-title">Top Rated Hotels{{ city ? ` In ${city.charAt(0).toUpperCase() + city.slice(1)}` : '' }}</h2>
        <div class="controls">
          <label class="rating-filter">
            <input type="checkbox" v-model="minRatingOnly" />
            8.0+ rated only
          </label>
          <div class="sort-control">
            <label for="hotel-sort">Sort by</label>
            <select id="hotel-sort" v-model="sortBy">
              <option value="price">Lowest Price</option>
              <option value="rating">Highest Rated</option>
            </select>
          </div>
        </div>
      </div>
  
      <div v-if="error" class="error-text">{{ error }}</div>
      <template v-else>
        <div class="hotel-scroll">
          <div v-for="hotel in sortedHotels" :key="hotel.hotel_id" class="hotel-card">
            <div class="hotel-image-wrap">
              <img
                v-if="hotel.photoUrl"
                :src="hotel.photoUrl"
                :alt="hotel.name"
                class="hotel-image"
                loading="lazy"
              />
              <div class="rating-badge" v-if="hotel.reviewCount > 0">
                ★ {{ hotel.reviewScore }}
                <span class="review-count">({{ hotel.reviewCount }})</span>
              </div>
            </div>
            <h3 class="hotel-name">{{ hotel.name }}</h3>
            <div class="hotel-footer">
              <div v-if="hotel.price">
                <div class="hotel-price">
                  {{ hotel.priceCurrency }} {{ hotel.price.toFixed(0) }}
                </div>
                <div class="per-night">per night</div>
              </div>
              <a
                :href="hotel.bookingUrl"
                target="_blank"
                rel="noopener"
                class="book-btn"
                >Book</a
              >
            </div>
          </div>
        </div>
        <HotelMap :hotels="sortedHotels" />
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
  .hotel-section {
    margin-bottom: 32px;
  }
  
  .section-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 12px;
    margin-bottom: 24px;
  }
  
  .section-title {
    font-family: var(--font-display);
    font-size: 24px;
    font-weight: 600;
    color: var(--color-primary);
  }
  
  .controls {
    display: flex;
    align-items: center;
    gap: 24px;
  }
  
  .rating-filter {
    display: flex;
    align-items: center;
    gap: 6px;
    font-size: 13px;
    color: var(--color-on-surface-variant);
    cursor: pointer;
  }
  
  .sort-control {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 13px;
    color: var(--color-on-surface-variant);
  }
  
  .sort-control select {
    font-family: var(--font-body);
    font-size: 13px;
    padding: 6px 10px;
    border: 1px solid var(--color-outline-variant);
    border-radius: var(--radius-md);
    background: white;
    cursor: pointer;
  }
  
  .hotel-scroll {
    display: flex;
    gap: 24px;
    overflow-x: auto;
    padding-bottom: 8px;
  }
  
  .hotel-card {
    flex: 0 0 auto;
    width: 260px;
  }
  
  .hotel-image-wrap {
    position: relative;
    height: 180px;
    border-radius: var(--radius-lg);
    overflow: hidden;
    margin-bottom: 12px;
  }
  
  .hotel-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
  
  .rating-badge {
    position: absolute;
    bottom: 10px;
    left: 10px;
    background: rgba(0, 0, 0, 0.55);
    color: white;
    padding: 4px 10px;
    border-radius: 999px;
    font-size: 13px;
    font-weight: 600;
  }
  
  .review-count {
    font-weight: 400;
    opacity: 0.85;
    font-size: 11px;
  }
  
  .hotel-name {
    font-size: 18px;
    font-weight: 600;
    color: var(--color-primary);
    margin: 0 0 12px;
  }
  
  .hotel-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .hotel-price {
    color: var(--color-primary-container);
    font-weight: 700;
    font-size: 18px;
  }
  
  .per-night {
    color: var(--color-on-surface-variant);
    font-size: 11px;
  }
  
  .book-btn {
    background: var(--color-primary-container);
    color: white;
    text-decoration: none;
    padding: 8px 24px;
    border-radius: var(--radius-md);
    font-size: 13px;
    font-weight: 600;
    transition: filter 0.15s;
  }
  
  .book-btn:hover {
    filter: brightness(1.1);
  }
  
  .error-text {
    color: var(--color-error);
  }
  </style>