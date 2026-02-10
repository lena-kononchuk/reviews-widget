<template>
  <div class="min-h-screen bg-gradient-to-br from-teal-50 via-blue-50 to-cyan-50 py-8">
    <div class="mx-auto max-w-4xl px-4 sm:px-6 lg:px-8">
      <!-- Header with logo and back button -->
      <header class="mb-8">
        <div class="flex items-center justify-between">
          <router-link to="/" class="flex items-center gap-3 text-gray-600 hover:text-gray-900 transition-colors">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" />
            </svg>
            <span class="font-medium">{{ $t('message.backButton') || 'Back' }}</span>
          </router-link>

          <router-link to="/">
            <img src="../assets/img/google.svg" alt="Google Logo" class="h-10" />
          </router-link>
        </div>
      </header>

      <!-- Title -->
      <div class="mb-8 text-center">
        <h1 class="text-3xl font-bold bg-gradient-to-r from-teal-600 to-cyan-600 bg-clip-text text-transparent mb-2">
          {{ $t('message.allReviews') || 'Customer Reviews' }}
        </h1>
        <p class="text-gray-600">
          {{ reviews.length }} {{ $t('message.reviewsCount') }}
        </p>
      </div>

      <!-- Loading/Error states -->
      <div v-if="loading" class="text-center py-12">
        <div class="inline-block animate-spin rounded-full h-12 w-12 border-b-2 border-teal-600"></div>
        <p class="mt-4 text-gray-600">{{ $t('message.loading') }}</p>
      </div>

      <div v-else-if="error" class="text-center py-12">
        <p class="text-red-600">{{ error }}</p>
      </div>

      <!-- Reviews grid -->
      <div v-else class="space-y-6">
        <div
          v-for="review in reviews"
          :key="review.id"
          class="bg-white rounded-xl p-6 shadow-lg hover:shadow-2xl transition-all duration-300 border border-gray-200"
        >
          <!-- Review header -->
          <div class="flex items-start gap-4 mb-4">
            <!-- Avatar or Initials -->
            <div class="relative w-14 h-14 flex-shrink-0">
              <img
                v-if="review.avatar && !imageErrors[review.id]"
                :src="review.avatar"
                alt="avatar"
                class="w-14 h-14 rounded-full object-cover border-2 border-gray-100"
                @error="handleImageError(review.id)"
              />
              <div
                v-else
                :style="{ backgroundColor: getColorForName(review.name) }"
                class="w-14 h-14 rounded-full flex items-center justify-center text-white font-bold text-lg"
              >
                {{ getInitials(review.name) }}
              </div>
            </div>

            <div class="flex-1">
              <h3 class="font-semibold text-gray-900 text-lg mb-1">{{ review.name }}</h3>
              <div v-if="review.rating && review.rating > 0" class="flex items-center gap-2">
                <div class="flex">
                  <span
                    v-for="i in 5"
                    :key="i"
                    class="text-2xl"
                    :class="i <= review.rating ? 'text-yellow-400' : 'text-gray-300'"
                  >★</span>
                </div>
                <span class="text-sm text-gray-500">{{ review.rating }}/5</span>
              </div>
            </div>
          </div>

          <!-- Review text -->
          <p v-if="review.reviews && !review.reviews.includes('Invalid faker')" class="text-gray-700 leading-relaxed mb-3">{{ review.reviews }}</p>

          <!-- Review date -->
          <p class="text-xs text-gray-400">{{ formatDate(review.createdAt) }}</p>
        </div>
      </div>

      <!-- Empty state -->
      <div v-if="!loading && !error && reviews.length === 0" class="text-center py-12">
        <p class="text-gray-500 text-lg">{{ $t('message.noReviews') || 'No reviews yet' }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const reviews = ref([]);
const loading = ref(true);
const error = ref(null);
const imageErrors = ref({});

// Fetch reviews from the API using axios
const fetchReviews = async () => {
  try {
    const response = await axios.get('https://67951133aad755a134eb1e31.mockapi.io/reviews');
    const data = response.data;

    // Filter out invalid reviews
    const validReviews = data.filter(review => {
      // Must have valid rating
      if (!review.rating || isNaN(parseFloat(review.rating)) || parseFloat(review.rating) === 0) {
        return false;
      }
      // Must have valid review text (not faker error)
      if (!review.reviews || review.reviews.includes('Invalid faker') || review.reviews.includes('random.numeric')) {
        return false;
      }
      return true;
    });

    // Sort reviews by creation date, last comment first
    reviews.value = validReviews.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));


    console.log(`Loaded ${validReviews.length} valid reviews (filtered ${data.length - validReviews.length} invalid)`);
  } catch (err) {
    error.value = 'Failed to load reviews';
  } finally {
    loading.value = false;
  }
};

// Handle image loading errors
const handleImageError = (reviewId) => {
  imageErrors.value[reviewId] = true;
};

// Get initials from name
const getInitials = (name) => {
  if (!name) return '?';
  const parts = name.trim().split(' ');
  if (parts.length >= 2) {
    return (parts[0][0] + parts[parts.length - 1][0]).toUpperCase();
  }
  return name.substring(0, 2).toUpperCase();
};

// Generate consistent color for name
const getColorForName = (name) => {
  const colors = [
    '#3B82F6', // blue
    '#10B981', // green
    '#8B5CF6', // purple
    '#F59E0B', // amber
    '#EF4444', // red
    '#EC4899', // pink
    '#6366F1', // indigo
    '#14B8A6', // teal
    '#F97316', // orange
    '#06B6D4', // cyan
  ];

  let hash = 0;
  for (let i = 0; i < name.length; i++) {
    hash = name.charCodeAt(i) + ((hash << 5) - hash);
  }
  return colors[Math.abs(hash) % colors.length];
};

// Format date to 'uk-UA' locale
const formatDate = (dateStr) => {
  const date = new Date(dateStr);
  const formattedDate = date.toLocaleDateString('uk-UA');
  const formattedTime = date.toLocaleTimeString('uk-UA', { hour: '2-digit', minute: '2-digit' });
  return `${formattedDate} ${formattedTime}`;
};

onMounted(() => {
  fetchReviews();
});
</script>
