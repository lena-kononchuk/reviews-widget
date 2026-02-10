<template>
  <div class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center z-50 p-4" @click="handleBackgroundClick">
    <div class="bg-white rounded-2xl max-w-md w-full shadow-2xl border border-gray-200" @click.stop>
      <!-- Header -->
      <div class="flex items-center justify-between p-6 border-b border-gray-100">
        <h2 class="text-2xl font-bold text-gray-900">{{ $t('message.writeReviewTitle') }}</h2>
        <button 
          @click="closeModal" 
          class="text-gray-400 hover:text-gray-600 transition-colors"
          aria-label="Close"
        >
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>

      <!-- Form -->
      <div class="p-6 space-y-4">
        <!-- Name input -->
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-2">
            {{ $t('message.yourName') || 'Your Name' }} <span class="text-red-500">*</span>
          </label>
          <input
            v-model="reviewerName"
            type="text"
            :placeholder="$t('message.namePlaceholder') || 'Enter your name'"
            class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-teal-500 focus:border-transparent transition-all"
          />
        </div>

        <!-- Rating -->
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-2">
            {{ $t('message.rating') || 'Rating' }} <span class="text-red-500">*</span>
          </label>
          <div class="flex gap-2">
            <button
              v-for="star in 5"
              :key="star"
              @click="rating = star"
              class="text-4xl transition-all hover:scale-110"
              :class="star <= rating ? 'text-yellow-400' : 'text-gray-300'"
            >
              ★
            </button>
          </div>
        </div>

        <!-- Review text -->
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-2">
            {{ $t('message.yourReview') || 'Your Review' }} <span class="text-red-500">*</span>
          </label>
          <textarea
            v-model="reviewText"
            :placeholder="$t('message.placeholderText')"
            rows="4"
            class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-teal-500 focus:border-transparent transition-all resize-none"
          ></textarea>
        </div>

        <!-- Error message -->
        <p v-if="errorMessage" class="text-red-500 text-sm">{{ errorMessage }}</p>
      </div>

      <!-- Footer -->
      <div class="flex gap-3 p-6 border-t border-gray-100">
        <button
          @click="closeModal"
          class="flex-1 px-4 py-3 text-gray-700 bg-gray-100 rounded-lg font-medium hover:bg-gray-200 transition-colors"
        >
          {{ $t('message.cancel') || 'Cancel' }}
        </button>
        <button
          @click="submitReview"
          :disabled="!isValid"
          :class="isValid ? 'bg-gradient-to-r from-teal-500 to-teal-600 hover:from-teal-600 hover:to-teal-700' : 'bg-gray-300 cursor-not-allowed'"
          class="flex-1 px-4 py-3 text-white rounded-lg font-medium transition-all"
        >
          {{ $t('message.submitButton') }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { defineEmits, ref, computed, onMounted, onBeforeUnmount } from 'vue';
import axios from 'axios';

const emit = defineEmits(['review-added', 'close-modal']);

const reviewerName = ref('');
const reviewText = ref('');
const rating = ref(0);
const errorMessage = ref('');

const isValid = computed(() => {
  return reviewerName.value.trim() !== '' && 
         reviewText.value.trim() !== '' && 
         rating.value > 0;
});

const submitReview = async () => {
  if (!isValid.value) {
    errorMessage.value = 'Please fill in all fields and select a rating';
    return;
  }

  try {
    const reviewUrl = `https://67951133aad755a134eb1e31.mockapi.io/reviews`;

    const response = await axios.post(reviewUrl, {
      name: reviewerName.value,
      reviews: reviewText.value,
      rating: rating.value,
      avatar: '', // No avatar - will show initials
    });

    // Reset form
    reviewerName.value = '';
    reviewText.value = '';
    rating.value = 0;
    errorMessage.value = '';

    emit('review-added', response.data);
    closeModal();
  } catch (error) {
    errorMessage.value = 'Failed to submit review. Please try again.';
    console.error('Error:', error.response ? error.response.data : error.message);
  }
};

const closeModal = () => {
  emit('close-modal');
};

const handleBackgroundClick = (e) => {
  if (e.target === e.currentTarget) {
    closeModal();
  }
};

onMounted(() => {
  const handleEsc = (e) => {
    if (e.key === 'Escape') {
      closeModal();
    }
  };
  window.addEventListener('keydown', handleEsc);

  onBeforeUnmount(() => {
    window.removeEventListener('keydown', handleEsc);
  });
});
</script>
