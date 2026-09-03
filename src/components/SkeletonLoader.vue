<template>
  <div class="skeleton-wrap" role="status" aria-live="polite">
    <span class="sr-only">Loading your trip results…</span>

    <!-- Trip cost summary -->
    <div class="skel-summary shimmer"></div>

    <!-- Flights: filter sidebar + row list -->
    <div class="skeleton-section">
      <div class="skel-title shimmer"></div>
      <div class="skel-split">
        <div class="skel-filters shimmer"></div>
        <div class="skel-rows">
          <div class="skel-row-card shimmer" v-for="n in 4" :key="n"></div>
        </div>
      </div>
    </div>

    <!-- Hotels: horizontal rail of cards -->
    <div class="skeleton-section">
      <div class="skel-title shimmer"></div>
      <div class="skel-rail">
        <div class="skel-card shimmer" v-for="n in 4" :key="n"></div>
      </div>
    </div>

    <!-- Weather: stacked day rows -->
    <div class="skeleton-section">
      <div class="skel-title shimmer"></div>
      <div class="skel-panel shimmer"></div>
    </div>
  </div>
</template>

<script setup>
</script>

<style scoped>
.skeleton-wrap {
  display: flex;
  flex-direction: column;
  gap: var(--space-8);
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* One shimmer definition, reused — this used to be copy-pasted onto five
   separate rules. */
.shimmer {
  background: linear-gradient(
    90deg,
    var(--color-surface-container-low) 25%,
    var(--color-surface-container) 50%,
    var(--color-surface-container-low) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.4s ease-in-out infinite;
}

.skel-summary {
  height: 11rem;
  border-radius: var(--radius-xl);
}

.skel-title {
  width: 13rem;
  height: 1.5rem;
  border-radius: var(--radius-sm);
  margin-bottom: var(--space-5);
}

.skel-split {
  display: grid;
  grid-template-columns: 14rem 1fr;
  gap: var(--space-5);
  align-items: start;
}

.skel-filters {
  height: 13rem;
  border-radius: var(--radius-lg);
}

.skel-rows {
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
}

.skel-row-card {
  height: 5.5rem;
  border-radius: var(--radius-lg);
}

.skel-rail {
  display: flex;
  gap: var(--space-4);
  overflow: hidden;
}

.skel-card {
  flex: 0 0 auto;
  width: 16.5rem;
  height: 17rem;
  border-radius: var(--radius-lg);
}

.skel-panel {
  height: 18rem;
  border-radius: var(--radius-lg);
}

@keyframes shimmer {
  0% {
    background-position: 200% 0;
  }
  100% {
    background-position: -200% 0;
  }
}

@media (max-width: 900px) {
  .skel-split {
    grid-template-columns: 1fr;
  }
}
</style>
