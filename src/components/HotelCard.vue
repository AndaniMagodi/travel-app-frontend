<template>
    <section class="hotel-section">
      <h2 class="section-title">Top Rated Hotels</h2>
      <div v-if="error" class="error-text">{{ error }}</div>
      <div v-else class="hotel-scroll">
        <div v-for="hotel in hotels" :key="hotel.hotel_id" class="hotel-card">
          <div class="hotel-image-wrap">
            <img v-if="hotel.photoUrl" :src="hotel.photoUrl" :alt="hotel.name" class="hotel-image" />
            <div class="rating-badge" v-if="hotel.reviewCount > 0">
              ★ {{ hotel.reviewScore }} <span class="review-count">({{ hotel.reviewCount }})</span>
            </div>
          </div>
          <h3 class="hotel-name">{{ hotel.name }}</h3>
          <div class="hotel-footer">
            <div v-if="hotel.price">
              <div class="hotel-price">{{ hotel.priceCurrency }} {{ hotel.price.toFixed(0) }}</div>
              <div class="per-night">per night</div>
            </div>
            <a :href="hotel.bookingUrl" target="_blank" rel="noopener" class="book-btn">Book</a>
          </div>
        </div>
      </div>
    </section>
  </template>
  
  <script setup>
  import { computed } from 'vue'
  
  const props = defineProps({
    hotelsResult: { type: Object, required: true },
  })
  
  const error = computed(() =>
    props.hotelsResult.status === 'error' ? props.hotelsResult.error : null
  )
  
  // CONFIRMED: grossPrice is the real current price (strikethroughPrice is the
  // pre-discount "was" price, not what the guest actually pays).
  //
  // bookingUrl points to Booking.com's public search results page, pre-filled
  // with this hotel's name and the searched dates — NOT a deep link to this
  // exact offer (Booking.com's API doesn't expose one without a separate,
  // more involved booking-flow integration). This continues the booking
  // journey on Booking.com's real site rather than completing it on ours.
  const hotels = computed(() => {
    if (props.hotelsResult.status !== 'ok') return []
    const rawHotels = props.hotelsResult.data.data.hotels || []
  
    return rawHotels.map((h) => {
      const property = h.property
      const priceInfo = property.priceBreakdown?.grossPrice
  
      const bookingUrl =
        `https://www.booking.com/searchresults.html` +
        `?ss=${encodeURIComponent(property.name)}` +
        `&checkin=${property.checkinDate}` +
        `&checkout=${property.checkoutDate}`
  
      return {
        hotel_id: h.hotel_id,
        name: property.name,
        reviewScore: property.reviewScore,
        reviewCount: property.reviewCount,
        photoUrl: property.photoUrls?.[0] || null,
        price: priceInfo?.value ?? null,
        priceCurrency: priceInfo?.currency ?? '',
        bookingUrl,
      }
    })
  })
  </script>
  
  <style scoped>
  .hotel-section {
    margin-bottom: 48px;
  }
  
  .section-title {
    font-family: var(--font-display);
    font-size: 24px;
    font-weight: 600;
    color: var(--color-primary);
    margin-bottom: 24px;
  }
  
  .hotel-scroll {
    display: flex;
    gap: 16px;
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
    padding: 8px 16px;
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