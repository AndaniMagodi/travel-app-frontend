<template>
    <div class="skeleton-wrap">
      <div class="skeleton-section">
        <div class="skel-title"></div>
        <div class="skel-row">
          <div class="skel-box" v-for="n in 7" :key="n"></div>
        </div>
      </div>
      <div class="skeleton-section">
        <div class="skel-title"></div>
        <div class="skel-row">
          <div class="skel-box wide" v-for="n in 4" :key="n"></div>
        </div>
      </div>
      <div class="skeleton-section">
        <div class="skel-title"></div>
        <div class="skel-line" v-for="n in 4" :key="n"></div>
      </div>
    </div>
  </template>
  
  <script setup>
  </script>
  
  <style scoped>
  .skeleton-wrap {
    display: flex;
    flex-direction: column;
    gap: 40px;
  }
  
  .skel-title {
    width: 180px;
    height: 22px;
    border-radius: 4px;
    margin-bottom: 20px;
    background: linear-gradient(90deg, #eee 25%, #f5f5f5 50%, #eee 75%);
    background-size: 200% 100%;
    animation: shimmer 1.4s infinite;
  }
  
  .skel-row {
    display: flex;
    gap: 16px;
    flex-wrap: wrap;
  }
  
  .skel-box {
    width: 110px;
    height: 110px;
    border-radius: 12px;
    background: linear-gradient(90deg, #eee 25%, #f5f5f5 50%, #eee 75%);
    background-size: 200% 100%;
    animation: shimmer 1.4s infinite;
  }
  
  .skel-box.wide {
    width: 240px;
    height: 180px;
  }
  
  .skel-line {
    height: 70px;
    border-radius: 12px;
    margin-bottom: 12px;
    background: linear-gradient(90deg, #eee 25%, #f5f5f5 50%, #eee 75%);
    background-size: 200% 100%;
    animation: shimmer 1.4s infinite;
  }
  
  @keyframes shimmer {
    0% { background-position: 200% 0; }
    100% { background-position: -200% 0; }
  }
  </style>