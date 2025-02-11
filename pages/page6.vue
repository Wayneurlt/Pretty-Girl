<template>
  <div class="min-h-screen flex items-center justify-center bg-indigo-100 font-sans">
    <div class="text-center p-8 bg-transparent rounded-2xl max-w-md mx-4">
      <img src="/assets/images/val.png" alt="Cute Illustration" class="mx-auto mb-6 w-32 h-32 object-cover" />

      <h1 class="text-4xl text-indigo-600 mb-6 animate-fade-in">Will you be my Valentine?</h1>
      
      <div class="flex justify-center gap-4 mt-6">
        <button
          @click="showForm = true"
          v-if="!showForm"
          class="px-6 py-3 bg-indigo-500 text-white rounded-lg shadow hover:bg-indigo-600 transition duration-300 transform hover:scale-105"
        >
          YES!
        </button>
        <button
          @click="moveNoButton"
          v-if="!showForm"
          :style="noButtonStyle"
          class="px-6 py-3 bg-indigo-500 text-white rounded-lg shadow hover:bg-indigo-600 transition duration-300 transform hover:scale-105 absolute"
        >
          NO!
        </button>
      </div>

      <!-- Form Modal -->
      <div v-if="showForm" class="animate-fade-in">
        <form @submit.prevent="handleSubmit" class="space-y-4 mt-6">
          <div class="space-y-2">
            <input
              v-model="formData.name"
              type="text"
              placeholder="Your Name"
              required
              class="w-full px-4 py-2 rounded-lg border border-indigo-300 focus:outline-none focus:ring-2 focus:ring-indigo-500"
              :class="{ 'border-red-500': errors.name }"
            />
            <p v-if="errors.name" class="text-red-500 text-sm text-left">{{ errors.name }}</p>
          </div>

          <div class="space-y-2">
            <input
              v-model="formData.email"
              type="email"
              placeholder="Your Email"
              required
              class="w-full px-4 py-2 rounded-lg border border-indigo-300 focus:outline-none focus:ring-2 focus:ring-indigo-500"
              :class="{ 'border-red-500': errors.email }"
            />
            <p v-if="errors.email" class="text-red-500 text-sm text-left">{{ errors.email }}</p>
          </div>

          <div class="flex gap-4 justify-center">
            <button
              type="submit"
              :disabled="isSubmitting"
              class="px-6 py-3 bg-indigo-500 text-white rounded-lg shadow hover:bg-indigo-600 transition duration-300 transform hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed"
            >
              {{ isSubmitting ? 'Submitting...' : 'Submit' }}
            </button>
            <button
              type="button"
              @click="showForm = false"
              class="px-6 py-3 bg-gray-500 text-white rounded-lg shadow hover:bg-gray-600 transition duration-300 transform hover:scale-105"
            >
              Cancel
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import emailjs from '@emailjs/browser';  // Import EmailJS

const router = useRouter();
const showForm = ref(false);
const isSubmitting = ref(false);

// Form Data
const formData = reactive({
  name: '',
  email: ''
});

// Error Handling
const errors = reactive({
  name: '',
  email: ''
});

// Button Movement Logic for "NO!"
const noButtonStyle = ref({
  position: 'static' as 'static' | 'absolute',
});

// EmailJS Configuration (Updated IDs)
const SERVICE_ID = 'service_pxu1wxz';
const TEMPLATE_ID = 'template_ds9f9fl';
const USER_ID = '4VE_fO_Bp6M16bG';

// Initialize EmailJS
onMounted(() => {
  emailjs.init(USER_ID);
});

// Email Validation
function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

// Form Validation Logic
function validateForm(): boolean {
  let isValid = true;
  errors.name = '';
  errors.email = '';

  if (!formData.name.trim()) {
    errors.name = 'Name is required';
    isValid = false;
  }

  if (!formData.email.trim()) {
    errors.email = 'Email is required';
    isValid = false;
  } else if (!validateEmail(formData.email)) {
    errors.email = 'Please enter a valid email';
    isValid = false;
  }

  return isValid;
}

// Submit Logic with Email Notification
async function handleSubmit() {
  if (!validateForm()) return;

  isSubmitting.value = true;

  try {
    // Sending Email Notification via EmailJS
    await emailjs.send(SERVICE_ID, TEMPLATE_ID, {
      from_name: formData.name,
      from_email: formData.email,
      message: 'Someone clicked YES to be your Valentine! ❤️'
    });

    // Optionally Store Data Locally
    localStorage.setItem('valentineResponse', JSON.stringify(formData));

    // Navigate to the next page on successful submission
    router.push('/page7');
  } catch (error) {
    console.error('Error submitting form:', error);
    alert('Oops! Something went wrong. Please try again later.');
  } finally {
    isSubmitting.value = false;
  }
}

// Move the "NO!" Button Randomly
function moveNoButton() {
  const x = Math.random() * window.innerWidth * 0.8;
  const y = Math.random() * window.innerHeight * 0.8;

  noButtonStyle.value = {
    position: 'absolute',
    left: `${x}px`,
    top: `${y}px`,
    transition: 'left 0.3s, top 0.3s',
  };
}
</script>

<style scoped>
@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-fade-in {
  animation: fade-in 1s ease-out;
}
</style>

<!-- Preload JSON resource with correct 'as' attribute -->
<link rel="preload" href="/_nuxt/builds/meta/9123aaf4-9569-4559-bf1f-7652b227247b.json" as="fetch" type="application/json" crossorigin="anonymous">
