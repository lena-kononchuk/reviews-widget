<template>
  <div class="inline-flex text-gray-300 text-3xl">
    <span
      v-for="i in 5"
      :key="i"
      :class="['star relative transition-all duration-200 ease-in-out', {
        'text-yellow-400': i <= fullStars,
        'text-gray-300': i > fullStars && i !== fullStars + 1,
      }]"
    >
      ★
      <span
        v-if="i === fullStars + 1 && hasHalfStar"
        class="absolute inset-0 w-1/2 text-yellow-400 overflow-hidden"
      >
        ★
      </span>
    </span>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  rating: {
    type: Number,
    default: 0
  }
});

// Make reactive computed properties
const fullStars = computed(() => Math.floor(props.rating));
const hasHalfStar = computed(() => props.rating - fullStars.value >= 0.5);
</script>
