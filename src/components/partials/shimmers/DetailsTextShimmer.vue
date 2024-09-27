<template>
    <div class="mt-1">
      <vue-skeleton-loader
        :width="120"
        :height="15"
      />
      <vue-skeleton-loader
        class="mt-1"
        :width="220"
        :height="12"
      />
    </div>
  </template>
  
  <script>
  import VueSkeletonLoader from 'skeleton-loader-vue'
  
  export default {
    components: {
      VueSkeletonLoader,
    },
  
  }
  </script>
  
  <style>
  
  </style>