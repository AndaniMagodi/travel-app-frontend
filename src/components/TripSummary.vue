<template>
  <div class="summary-card" v-if="hasEnoughData">
    <div class="summary-header">
      <div class="summary-heading">
        <span class="summary-eyebrow">Estimated trip cost</span>
        <span class="summary-sublabel">Lowest option in each category</span>
      </div>
      <div class="summary-amount">
        <span class="summary-currency">{{ currency }}</span>
        <span class="summary-total">{{ formatMoney(total) }}</span>
      </div>
    </div>

    <div class="summary-breakdown">
      <div class="line-item" v-if="cheapestFlightInfo">
        <span class="line-icon" aria-hidden="true">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7"
            stroke-linecap="round" stroke-linejoin="round">
            <path d="M2 14l20-7-7 20-3.5-7.5L2 14z" />
          </svg>
        </span>
        <span class="line-label">Flight</span>
        <span class="line-detail">{{ cheapestFlightInfo.airline }}</span>
        <span class="line-amount">{{ formatMoney(cheapestFlightInfo.amount) }}</span>
      </div>

      <div class="line-item" v-if="cheapestHotelInfo">
        <span class="line-icon" aria-hidden="true">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7"
            stroke-linecap="round" stroke-linejoin="round">
            <path d="M3 21V5.5A1.5 1.5 0 0 1 4.5 4h9A1.5 1.5 0 0 1 15 5.5V21" />
            <path d="M15 10h4.5A1.5 1.5 0 0 1 21 11.5V21M2 21h20" />
            <path d="M7 8h4M7 12h4M7 16h4" />
          </svg>
        </span>
        <span class="line-label">Stay</span>
        <span class="line-detail">
          {{ cheapestHotelInfo.name }}
          <span class="line-math">
            {{ nights }} × {{ formatMoney(cheapestHotelInfo.perNight) }}
          </span>
        </span>
        <span class="line-amount">{{ formatMoney(cheapestHotelInfo.perNight * nights) }}</span>
      </div>

      <div class="line-item" v-if="cheapestCarInfo">
        <span class="line-icon" aria-hidden="true">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7"
            stroke-linecap="round" stroke-linejoin="round">
            <path d="M4 16v-3l1.8-4.6A2 2 0 0 1 7.7 7h8.6a2 2 0 0 1 1.9 1.4L20 13v3" />
            <path d="M3 16h18v2a1 1 0 0 1-1 1h-1.5M3 16v2a1 1 0 0 0 1 1h1.5" />
            <circle cx="6.5" cy="16.5" r="1.6" />
            <circle cx="17.5" cy="16.5" r="1.6" />
          </svg>
        </span>
        <span class="line-label">Car</span>
        <span class="line-detail">
          {{ cheapestCarInfo.name }}
          <span class="line-math">
            {{ nights }} × {{ formatMoney(cheapestCarInfo.perDay) }}
          </span>
        </span>
        <span class="line-amount">{{ formatMoney(cheapestCarInfo.perDay * nights) }}</span>
      </div>
    </div>

    <p class="summary-note">
      Built from the lowest available price in each category, not a specific
      selected itinerary. Actual cost will vary.
    </p>
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

// Thousands separators — a four-digit total reads as a price, not a code.
function formatMoney(value) {
  return Math.round(value).toLocaleString('en-ZA')
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
</script>

<style scoped>
/* Deliberately NOT sticky. It used to be position:sticky at z-index 1000,
   which parked a 245px navy slab over the top quarter of the viewport for
   the entire scroll and covered the hotel map. */
.summary-card {
  background:
    radial-gradient(120% 140% at 100% 0%, rgba(27, 79, 134, 0.55) 0%, rgba(0, 30, 64, 0) 55%),
    linear-gradient(160deg, var(--navy-900) 0%, var(--navy-950) 100%);
  color: white;
  border-radius: var(--radius-xl);
  padding: var(--space-5) var(--space-6);
  margin-bottom: var(--space-8);
  box-shadow: var(--shadow-navy);
}

.summary-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  flex-wrap: wrap;
  gap: var(--space-4);
  padding-bottom: var(--space-5);
  border-bottom: 1px solid rgba(255, 255, 255, 0.16);
}

.summary-heading {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
}

.summary-eyebrow {
  font-family: var(--font-display);
  font-size: var(--text-lg);
  font-weight: 600;
  letter-spacing: -0.01em;
}

.summary-sublabel {
  font-size: var(--text-xs);
  letter-spacing: var(--tracking-caps);
  text-transform: uppercase;
  color: rgba(255, 255, 255, 0.62);
}

.summary-amount {
  display: flex;
  align-items: baseline;
  gap: var(--space-2);
}

.summary-currency {
  font-size: var(--text-sm);
  font-weight: 600;
  letter-spacing: var(--tracking-caps);
  color: rgba(255, 255, 255, 0.66);
}

.summary-total {
  font-size: var(--text-2xl);
  font-weight: 700;
  letter-spacing: -0.02em;
  /* Tabular figures so the total doesn't shift width between searches. */
  font-variant-numeric: tabular-nums;
}

.summary-breakdown {
  display: flex;
  flex-direction: column;
  padding-top: var(--space-2);
}

.line-item {
  display: grid;
  grid-template-columns: 1.75rem 3.25rem 1fr auto;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-3) 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}

.line-item:last-child {
  border-bottom: none;
}

.line-icon {
  display: grid;
  place-items: center;
  width: 1.75rem;
  height: 1.75rem;
  border-radius: var(--radius-sm);
  background: rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.9);
}

.line-icon svg {
  width: 15px;
  height: 15px;
}

.line-label {
  font-size: var(--text-xs);
  font-weight: 600;
  letter-spacing: var(--tracking-caps);
  text-transform: uppercase;
  color: rgba(255, 255, 255, 0.62);
}

.line-detail {
  font-size: var(--text-sm);
  color: rgba(255, 255, 255, 0.95);
  min-width: 0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.line-math {
  color: rgba(255, 255, 255, 0.55);
  font-variant-numeric: tabular-nums;
}

.line-amount {
  font-size: var(--text-base);
  font-weight: 600;
  font-variant-numeric: tabular-nums;
  text-align: right;
}

.summary-note {
  margin-top: var(--space-4);
  font-size: var(--text-xs);
  line-height: 1.5;
  color: rgba(255, 255, 255, 0.55);
  max-width: 34rem;
}

@media (max-width: 640px) {
  .summary-card {
    padding: var(--space-5) var(--space-4);
  }

  .line-item {
    grid-template-columns: 1.75rem 1fr auto;
  }

  .line-label {
    display: none;
  }

  .line-detail {
    white-space: normal;
  }
}
</style>
