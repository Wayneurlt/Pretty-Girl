<template>
  <div class="min-h-screen flex items-center justify-center bg-indigo-100 font-sans">
    <div class="text-center p-8 bg-transparent rounded-2xl max-w-md mx-4">
      <img 
        src="/assets/images/val.png" 
        alt="Valentine's Day Illustration" 
        class="mx-auto mb-6 w-32 h-32 object-cover" 
      />

      <h1 class="text-4xl text-indigo-600 mb-6 animate-fade-in">
        Will you be my Valentine?
      </h1>
      
      <div class="flex justify-center gap-4 mt-6">
        <button
          v-if="!showForm"
          @click="showForm = true"
          class="px-6 py-3 bg-indigo-500 text-white rounded-lg shadow hover:bg-indigo-600 transition duration-300 transform hover:scale-105"
          aria-label="Accept Valentine's request"
        >
          YES!
        </button>
        <button
          v-if="!showForm"
          @click="moveNoButton"
          :style="noButtonStyle"
          class="px-6 py-3 bg-indigo-500 text-white rounded-lg shadow hover:bg-indigo-600 transition duration-300 transform hover:scale-105 absolute"
          aria-label="Decline Valentine's request"
          ref="noButton"
        >
          NO!
        </button>
      </div>

      <Transition name="fade">
        <div v-if="showForm" class="animate-fade-in">
          <form @submit.prevent="handleSubmit" class="space-y-4 mt-6">
            <div class="space-y-2">
              <input
                v-model="formData.name"
                type="text"
                placeholder="Your Name"
                required
                class="w-full px-4 py-2 rounded-lg border border-indigo-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white text-black"
                :class="{ 'border-red-500': errors.name }"
                aria-label="Your name"
                :aria-invalid="!!errors.name"
                :aria-describedby="errors.name ? 'name-error' : undefined"
              />
              <p 
                v-if="errors.name" 
                id="name-error"
                class="text-red-500 text-sm text-left"
                role="alert"
              >
                {{ errors.name }}
              </p>
            </div>

            <div class="space-y-2">
              <input
                v-model="formData.email"
                type="email"
                placeholder="Your Email"
                required
                class="w-full px-4 py-2 rounded-lg border border-indigo-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white text-black"
                :class="{ 'border-red-500': errors.email }"
                aria-label="Your email"
                :aria-invalid="!!errors.email"
                :aria-describedby="errors.email ? 'email-error' : undefined"
              />
              <p 
                v-if="errors.email" 
                id="email-error"
                class="text-red-500 text-sm text-left"
                role="alert"
              >
                {{ errors.email }}
              </p>
            </div>

            <div class="flex gap-4 justify-center">
              <button
                type="submit"
                :disabled="isSubmitting"
                class="px-6 py-3 bg-indigo-500 text-white rounded-lg shadow hover:bg-indigo-600 transition duration-300 transform hover:scale-105 disabled:opacity-50 disabled:cursor-not-allowed"
                aria-label="Submit Valentine's response"
              >
                <span v-if="isSubmitting" class="flex items-center gap-2">
                  <span class="animate-spin">↻</span>
                  Submitting...
                </span>
                <span v-else>Submit</span>
              </button>
              <button
                type="button"
                @click="cancelForm"
                class="px-6 py-3 bg-gray-500 text-white rounded-lg shadow hover:bg-gray-600 transition duration-300 transform hover:scale-105"
                aria-label="Cancel form submission"
              >
                Cancel
              </button>
            </div>
          </form>
        </div>
      </Transition>

      <!-- Success/Error Toast Messages -->
      <Transition name="fade">
        <div 
          v-if="notification.show" 
          :class="[
            'fixed bottom-4 right-4 p-4 rounded-lg shadow-lg',
            notification.type === 'success' ? 'bg-green-500' : 'bg-red-500',
            'text-white'
          ]"
          role="alert"
        >
          {{ notification.message }}
        </div>
      </Transition>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import emailjs from '@emailjs/browser';

// Types
interface FormData {
  name: string;
  email: string;
}

interface FormErrors {
  name: string;
  email: string;
}

interface ButtonStyle {
  position: 'static' | 'absolute';
  left?: string;
  top?: string;
  transition?: string;
}

interface Notification {
  show: boolean;
  message: string;
  type: 'success' | 'error';
}

// Environment variables (should be in .env file)
const EMAIL_CONFIG = {
  SERVICE_ID: import.meta.env.VITE_EMAILJS_SERVICE_ID,
  TEMPLATE_ID: import.meta.env.VITE_EMAILJS_TEMPLATE_ID,
  USER_ID: import.meta.env.VITE_EMAILJS_USER_ID,
} as const;

// State
const router = useRouter();
const showForm = ref(false);
const isSubmitting = ref(false);
const noButton = ref<HTMLButtonElement | null>(null);

const formData = reactive<FormData>({
  name: '',
  email: '',
});

const errors = reactive<FormErrors>({
  name: '',
  email: '',
});

const noButtonStyle = ref<ButtonStyle>({
  position: 'static',
});

const notification = reactive<Notification>({
  show: false,
  message: '',
  type: 'success',
});

// Initialization
onMounted(() => {
  if (EMAIL_CONFIG.USER_ID) {
    emailjs.init(EMAIL_CONFIG.USER_ID);
  } else {
    console.error('EmailJS configuration is missing');
  }
});

// Email Validation
const validateEmail = (email: string): boolean => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

// Form Validation
const validateForm = (): boolean => {
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
};

// Notification Handler
const showNotification = (message: string, type: 'success' | 'error') => {
  notification.show = true;
  notification.message = message;
  notification.type = type;

  setTimeout(() => {
    notification.show = false;
  }, 3000);
};

// Form Submission
const handleSubmit = async () => {
  if (!validateForm()) return;

  isSubmitting.value = true;

  try {
    if (!EMAIL_CONFIG.SERVICE_ID || !EMAIL_CONFIG.TEMPLATE_ID || !EMAIL_CONFIG.USER_ID) {
      throw new Error('EmailJS configuration is missing');
    }

    await emailjs.send(
      EMAIL_CONFIG.SERVICE_ID,
      EMAIL_CONFIG.TEMPLATE_ID,
      {
        from_name: formData.name,
        from_email: formData.email,
        message: 'Someone clicked YES to be your Valentine! ❤️',
      }
    );

    // Store response locally
    localStorage.setItem('valentineResponse', JSON.stringify({
      name: formData.name,
      email: formData.email,
      timestamp: new Date().toISOString(),
    }));

    showNotification('Thank you for your response! ❤️', 'success');
    
    // Navigate after a brief delay to show the success message
    setTimeout(() => {
      router.push('/page7');
    }, 1500);
  } catch (error) {
    console.error('Error submitting form:', error);
    showNotification('Oops! Something went wrong. Please try again.', 'error');
  } finally {
    isSubmitting.value = false;
  }
};

// Cancel Form
const cancelForm = () => {
  showForm.value = false;
  formData.name = '';
  formData.email = '';
  errors.name = '';
  errors.email = '';
};

// Move NO Button
const moveNoButton = () => {
  if (!noButton.value) return;

  const buttonRect = noButton.value.getBoundingClientRect();
  const maxX = window.innerWidth - buttonRect.width;
  const maxY = window.innerHeight - buttonRect.height;
  
  const x = Math.max(0, Math.min(Math.random() * maxX, maxX));
  const y = Math.max(0, Math.min(Math.random() * maxY, maxY));

  noButtonStyle.value = {
    position: 'absolute',
    left: `${x}px`,
    top: `${y}px`,
    transition: 'left 0.3s, top 0.3s',
  };
};
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

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