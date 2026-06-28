<template>
    <div class="summary-card" v-if="hasEnoughData">
      <div class="summary-header">
        <span class="summary-label">
          Estimated Trip Cost <span class="summary-sublabel">(lowest per category)</span>
        </span>
        <span class="summary-total">{{ currency }} {{ total.toFixed(0) }}</span>
      </div>
  
      <div class="summary-breakdown">
        <div class="line-item" v-if="cheapestFlightInfo">
          <div class="line-main">
            <span>✈️ Flight — {{ cheapestFlightInfo.airline }}</span>
            <span>{{ currency }} {{ cheapestFlightInfo.amount.toFixed(0) }}</span>
          </div>
          <div class="proportion-bar">
            <div class="proportion-fill" :style="{ width: pct(cheapestFlightInfo.amount) + '%' }"></div>
          </div>
        </div>
  
        <div class="line-item" v-if="cheapestHotelInfo">
          <div class="line-main">
            <span>🏨 {{ cheapestHotelInfo.name }} — {{ nights }} {{ nights === 1 ? 'night' : 'nights' }} × {{ currency }} {{ cheapestHotelInfo.perNight.toFixed(0) }}</span>
            <span>{{ currency }} {{ (cheapestHotelInfo.perNight * nights).toFixed(0) }}</span>
          </div>
          <div class="proportion-bar">
            <div class="proportion-fill" :style="{ width: pct(cheapestHotelInfo.perNight * nights) + '%' }"></div>
          </div>
        </div>
  
        <div class="line-item" v-if="cheapestCarInfo">
          <div class="line-main">
            <span>🚗 {{ cheapestCarInfo.name }} — {{ nights }} {{ nights === 1 ? 'day' : 'days' }} × {{ currency }} {{ cheapestCarInfo.perDay.toFixed(0) }}</span>
            <span>{{ currency }} {{ (cheapestCarInfo.perDay * nights).toFixed(0) }}</span>
          </div>
          <div class="proportion-bar">
            <div class="proportion-fill" :style="{ width: pct(cheapestCarInfo.perDay * nights) + '%' }"></div>
          </div>
        </div>
      </div>
  
      <p class="summary-note">Based on the lowest available price in each category. Actual cost may vary.</p>
    </div>
  </template>
  
  <script setup>
  import { computed } from 'vue'
  
  const props = defineProps({
    tripData: { type: Object, required: true },
  })
  
  const currency = 'ZAR'
  
  function moneyAmount({ currencyCode, units, nanos }) {
    return { currencyCode, amount: units + nanos / 1e9 }
  }
  
  const nights = computed(() => {
    const params = props.tripData?._searchParams
    if (!params?.checkIn || !params?.checkOut) return 1
    const checkin = new Date(params.checkIn)
    const checkout = new Date(params.checkOut)
    return Math.max(1, Math.round((checkout - checkin) / (1000 * 60 * 60 * 24)))
  })
  
  const cheapestFlightInfo = computed(() => {
    const offers = props.tripData?.flights?.data?.data?.flightOffers
    if (!offers?.length) return null
  
    let best = null
    for (const offer of offers) {
      const { amount } = moneyAmount(offer.priceBreakdown.total)
      if (!best || amount < best.amount) {
        const airline = offer.segments?.[0]?.legs?.[0]?.carriersData?.[0]?.name ?? 'Unknown airline'
        best = { amount, airline }
      }
    }
    return best
  })
  
  const cheapestHotelInfo = computed(() => {
    const hotels = props.tripData?.hotels?.data?.data?.hotels
    if (!hotels?.length) return null
    const rate = props.tripData?.exchangeRate?.usdToZar
  
    let best = null
    for (const h of hotels) {
      const info = h.property?.priceBreakdown?.grossPrice
      if (!info) continue
  
      const checkin = new Date(h.property.checkinDate)
      const checkout = new Date(h.property.checkoutDate)
      const stayNights = Math.max(1, Math.round((checkout - checkin) / (1000 * 60 * 60 * 24)))
  
      const totalForStay = info.currency === 'USD' && rate ? info.value * rate : info.value
      const perNight = totalForStay / stayNights
  
      if (!best || perNight < best.perNight) {
        best = { perNight, name: h.property.name }
      }
    }
    return best
  })
  
  const cheapestCarInfo = computed(() => {
    const cars = props.tripData?.carRental?.data?.cars
    if (!cars?.length) return null
  
    let best = null
    for (const c of cars) {
      if (!best || c.pricePerDay < best.perDay) {
        best = { perDay: c.pricePerDay, name: `${c.make} ${c.model}` }
      }
    }
    return best
  })
  
  const hasEnoughData = computed(
    () => cheapestFlightInfo.value || cheapestHotelInfo.value || cheapestCarInfo.value
  )
  
  const total = computed(() => {
    let sum = 0
    if (cheapestFlightInfo.value) sum += cheapestFlightInfo.value.amount
    if (cheapestHotelInfo.value) sum += cheapestHotelInfo.value.perNight * nights.value
    if (cheapestCarInfo.value) sum += cheapestCarInfo.value.perDay * nights.value
    return sum
  })
  
  function pct(amount) {
    if (!total.value) return 0
    return Math.min(100, (amount / total.value) * 100)
  }
  </script>
  
  <style scoped>
  .summary-card {
    position: sticky;
    top: 16px;
    z-index: 1000;
    background: var(--color-primary);
    color: white;
    border-radius: var(--radius-lg);
    padding: 24px 28px;
    margin-bottom: 32px;
  }
  
  .summary-header {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
    margin-bottom: 16px;
  }
  
  .summary-label {
    font-family: var(--font-display);
    font-size: 18px;
    font-weight: 600;
  }
  
  .summary-sublabel {
    font-size: 12px;
    font-weight: 400;
    opacity: 0.7;
  }
  
  .summary-total {
    font-size: 28px;
    font-weight: 700;
  }
  
  .summary-breakdown {
    display: flex;
    flex-direction: column;
    gap: 14px;
    border-top: 1px solid rgba(255, 255, 255, 0.2);
    padding-top: 16px;
  }
  
  .line-item {
    display: flex;
    flex-direction: column;
    gap: 6px;
  }
  
  .line-main {
    display: flex;
    justify-content: space-between;
    font-size: 13px;
    opacity: 0.95;
    gap: 12px;
  }
  
  .proportion-bar {
    height: 4px;
    background: rgba(255, 255, 255, 0.15);
    border-radius: 999px;
    overflow: hidden;
  }
  
  .proportion-fill {
    height: 4px;
    background: rgba(255, 255, 255, 0.85);
    border-radius: 999px;
  }
  
  .summary-note {
    margin: 16px 0 0;
    font-size: 11px;
    opacity: 0.7;
  }
  </style>