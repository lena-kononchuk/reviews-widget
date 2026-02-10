<template>
  <div class="flex items-center justify-center min-h-screen relative py-20 px-3 bg-gradient-to-br from-teal-50 via-blue-50 to-cyan-50">
    <!-- Wrapper for the entire component -->
    <div class="container bg-white p-7 rounded-xl shadow-xl border border-gray-100 animate-fade-in">
      <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between">
      <div class="flex flex-col md:flex-row mb-3 sm:mb-0">
        <!-- Content loading or error handling -->
        <div v-if="loading">{{ t('message.loading') }}</div>
        <div v-else-if="error">{{ error }}</div>
        <div v-else class="">
          <!-- Review data -->
          <div class="lg:flex lg:gap-5 xl:gap-14">
            <div class="flex gap-5 items-center sm:mb-3 xl:gap-5 lg:mb-0">
              <span><img src="/src/assets/img/google.svg" alt="logo" /></span>
                  <span class="flex-wrap font-formular  font-medium sm:text-xl sm:font-normal  sm:leading-[30px] text-[13.32px] leading-[20px]">
                      {{ t('message.reviewsTitle') }}
                  </span>
            </div>
            <div>
              <router-link to="/reviews" class="flex flex-col sm:flex-row sm:mb-3 sm:items-center sm:gap-3  lg:mb-0">
                <div class="flex gap-5 items-center">
                  <p  class="font-greenwich-medium text-[24px] leading-[29.33px] font-normal sm:text-[36px] sm:leading-[44px] sm:font-medium">{{ averageRating ? averageRating.toFixed(1) : '0.0' }}</p>
                  <div>
                    <StarRating :rating="averageRating" />
                  </div>
                </div>
                <p class="font-formular font-normal text-gray-400 text-sm font-normal decoration-0 xs:mb-3 sm:mb-0 pt-2">
                  {{ totalReviews }}
                  {{ t('message.reviewsCount', { count: totalReviews }) }}
                </p>
              </router-link>
            </div>
          </div>
        </div>
      </div>

      <!-- Buttons for viewing and writing reviews -->
      <div class="flex flex-col sm:flex-row xs:gap-2 text-center">
        <router-link to="/reviews" class="h-11 min-w-[120px] mb-2 sm:mb-0 px-4 py-3 text-sm inline-block text-center text-black bg-gradient-to-r from-[#E6F7F5] to-[#F8FCFC] border border-[#AACEDB] rounded-lg hover:bg-gradient-to-r hover:from-teal-600 hover:via-teal-500 hover:to-teal-400 transition duration-300 ease-in-out">
          {{ t('message.viewButton') }}
        </router-link>
        <button @click="openModal" class="h-11 min-w-[120px] px-4 py-2 text-sm  text-center text-white inline-block bg-gradient-to-r from-[#3CB9A0] to-[#1786AC] border rounded-lg hover:from-teal-400 hover:via-teal-500 hover:to-teal-600 transition duration-300 ease-in-out">
          {{ t('message.writeButton') }}
        </button>
      </div>
    </div>
    </div>
    <!-- Modal for adding reviews -->
    <teleport to="body">
      <div v-if="showModal" id="review-modal" @click.self="closeModal">
        <ReviewNew @close-modal="closeModal" @review-added="handleReviewAdded" />
      </div>
    </teleport>

    <!-- Language switcher -->
    <div class="absolute top-6 right-6 flex gap-2">
      <button
        @click="changeLanguage('uk')"
        :class="locale === 'uk' ? 'bg-teal-600 text-white' : 'bg-gray-200 text-gray-700'"
        class="px-4 py-2 rounded-lg font-medium transition-colors hover:opacity-80"
      >
        UK
      </button>
      <button
        @click="changeLanguage('en')"
        :class="locale === 'en' ? 'bg-teal-600 text-white' : 'bg-gray-200 text-gray-700'"
        class="px-4 py-2 rounded-lg font-medium transition-colors hover:opacity-80"
      >
        EN
      </button>
    </div>
  </div>
</template>

<script setup>
// Import necessary modules
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import ReviewNew from './ReviewNew.vue';
import StarRating from './StarRating.vue';
import { useI18n } from 'vue-i18n';

// Initialize i18n
const { t, locale } = useI18n();

// Reactive state variables
const totalReviews = ref(0);
const averageRating = ref(0);
const loading = ref(true);
const error = ref(null);
const showModal = ref(false);

// Function to open the modal
const openModal = () => {
  showModal.value = true;
};

// Function to close the modal
const closeModal = () => {
  showModal.value = false;
};

// Fetch review data from an API
const fetchData = async () => {
  try {
    const response = await axios.get('https://67951133aad755a134eb1e31.mockapi.io/reviews'); // API URL
    const data = response.data;

    // Filter reviews with valid ratings
    const validReviews = data.filter(review => review.rating && !isNaN(parseFloat(review.rating)));

    totalReviews.value = validReviews.length; // Set the total number of reviews

    if (validReviews.length > 0) {
      averageRating.value = validReviews.reduce((sum, review) => sum + parseFloat(review.rating), 0) / validReviews.length;
    } else {
      averageRating.value = 0;
    }

    console.log('Reviews loaded:', validReviews.length, 'Average:', averageRating.value);
  } catch (err) {
    error.value = 'Failed to load data'; // Set error message
    console.error('Error fetching data:', err);
  } finally {
    loading.value = false; // Set loading to false
  }
};

// Fetch data when the component is mounted
onMounted(() => {
  fetchData();
});

// Computed property for filled stars
const filledStars = computed(() => averageRating.value);

// Function to handle a new review being added
const handleReviewAdded = (newReview) => {
  console.log('Added review:', newReview);
  fetchData();
};

// Function to change the application language
const changeLanguage = (lang) => {
  locale.value = lang;
};
</script>
