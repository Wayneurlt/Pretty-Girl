<template>
  <div class="min-h-screen flex items-center justify-center bg-indigo-100 font-sans">
    <div class="text-center p-8 bg-transparent rounded-2xl max-w-md mx-4">
      <img 
        src="/assets/images/val.png" 
        alt="Valentine's Day Illustration" 
        class="mx-auto mb-6 w-32 h-32 object-cover"
      />

      <h1 class="text-4xl text-indigo-600 mb-6">
        Will you be my Valentine?
      </h1>
      
      <!-- Buttons Container -->
      <div v-if="!formVisible" class="flex justify-center gap-4 mt-6 relative">
        <button
          @click="handleYesClick"
          class="px-6 py-3 bg-indigo-500 text-white rounded-lg shadow hover:bg-indigo-600 transition duration-300 transform hover:scale-105"
        >
          YES!
        </button>
        
        <button
          @click="moveNoButton"
          :style="noButtonStyle"
          class="px-6 py-3 bg-indigo-500 text-white rounded-lg shadow hover:bg-indigo-600 transition duration-300 transform hover:scale-105"
          ref="noButtonRef"
        >
          NO!
        </button>
      </div>

      <!-- Toast Notification -->
      <div 
        v-if="notification.show"
        class="fixed bottom-4 right-4 p-4 rounded-lg shadow-lg text-white"
        :class="notification.type === 'success' ? 'bg-green-500' : 'bg-red-500'"
      >
        {{ notification.message }}
      </div>
    </div>

    <!-- Modal Popup for Form -->
    <div v-if="formVisible" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white p-8 rounded-2xl shadow-lg w-full max-w-md">
        <h2 class="text-2xl font-semibold text-indigo-600 mb-4 text-center">Fill in your details</h2>
        <form @submit.prevent="handleSubmit" class="space-y-4">
          <!-- Name Input -->
          <div class="space-y-2">
            <input
              v-model="formData.name"
              type="text"
              placeholder="Your Name"
              required
              class="w-full px-4 py-2 rounded-lg border border-indigo-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white text-black"
              :class="{ 'border-red-500': errors.name }"
            />
            <p v-if="errors.name" class="text-red-500 text-sm text-left">
              {{ errors.name }}
            </p>
          </div>

          <!-- Email Input -->
          <div class="space-y-2">
            <input
              v-model="formData.email"
              type="email"
              placeholder="Your Email"
              required
              class="w-full px-4 py-2 rounded-lg border border-indigo-300 focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white text-black"
              :class="{ 'border-red-500': errors.email }"
            />
            <p v-if="errors.email" class="text-red-500 text-sm text-left">
              {{ errors.email }}
            </p>
          </div>

          <!-- Form Buttons -->
          <div class="flex gap-4 justify-center">
            <button
              type="submit"
              :disabled="isSubmitting"
              class="px-6 py-3 bg-indigo-500 text-white rounded-lg shadow hover:bg-indigo-600 transition duration-300 transform hover:scale-105 disabled:opacity-50"
            >
              <span v-if="isSubmitting">Sending...</span>
              <span v-else>Submit</span>
            </button>
            
            <button
              type="button"
              @click="handleCancel"
              class="px-6 py-3 bg-gray-500 text-white rounded-lg shadow hover:bg-gray-600 transition duration-300"
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
import { ref, reactive, onMounted, nextTick } from 'vue';
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

interface Notification {
  show: boolean;
  message: string;
  type: 'success' | 'error';
}

// Router setup
const router = useRouter();

// Refs and reactive state
const formVisible = ref(false);
const isSubmitting = ref(false);
const noButtonRef = ref<HTMLButtonElement | null>(null);

const formData = reactive<FormData>({
  name: '',
  email: ''
});

const errors = reactive<FormErrors>({
  name: '',
  email: ''
});

const notification = reactive<Notification>({
  show: false,
  message: '',
  type: 'success'
});

const noButtonStyle = reactive({
  position: 'static' as const,
  left: '0px',
  top: '0px',
  transition: 'all 0.3s ease'
});

// Email configuration
const EMAIL_CONFIG = {
  SERVICE_ID: import.meta.env.VITE_EMAILJS_SERVICE_ID,
  TEMPLATE_ID: import.meta.env.VITE_EMAILJS_TEMPLATE_ID,
  USER_ID: import.meta.env.VITE_EMAILJS_USER_ID
};

// Initialize EmailJS
onMounted(() => {
  if (EMAIL_CONFIG.USER_ID) {
    emailjs.init(EMAIL_CONFIG.USER_ID);
  }
});

// Handle YES button click
const handleYesClick = async () => {
  console.log('YES button clicked');
  formVisible.value = true;
  await nextTick();  // Ensure DOM update is complete
};

// Handle form cancellation
const handleCancel = () => {
  formVisible.value = false;
  formData.name = '';
  formData.email = '';
  errors.name = '';
  errors.email = '';
};

// Validate email format
const validateEmail = (email: string): boolean => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

// Validate form
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

// Show notification
const showNotification = (message: string, type: 'success' | 'error') => {
  notification.show = true;
  notification.message = message;
  notification.type = type;

  setTimeout(() => {
    notification.show = false;
  }, 3000);
};

// Handle form submission
const handleSubmit = async () => {
  if (!validateForm()) return;

  isSubmitting.value = true;

  try {
    if (!EMAIL_CONFIG.SERVICE_ID || !EMAIL_CONFIG.TEMPLATE_ID) {
      throw new Error('Email configuration missing');
    }

    await emailjs.send(
      EMAIL_CONFIG.SERVICE_ID,
      EMAIL_CONFIG.TEMPLATE_ID,
      {
        from_name: formData.name,
        from_email: formData.email,
        message: 'Someone said YES to be your Valentine! ❤️'
      }
    );

    localStorage.setItem('valentineResponse', JSON.stringify({
      name: formData.name,
      email: formData.email,
      timestamp: new Date().toISOString()
    }));

    showNotification('Thanks for your response! ❤️', 'success');

    setTimeout(() => {
      router.push('/page7');
    }, 1500);
  } catch (error) {
    console.error('Submission error:', error);
    showNotification('Something went wrong. Please try again.', 'error');
  } finally {
    isSubmitting.value = false;
  }
};

// Move NO button
const moveNoButton = () => {
  const maxX = window.innerWidth - 100;
  const maxY = window.innerHeight - 100;

  const randX = Math.floor(Math.random() * maxX);
  const randY = Math.floor(Math.random() * maxY);

  Object.assign(noButtonStyle, {
    position: 'fixed',
    left: `${randX}px`,
    top: `${randY}px`
  });
};
</script>

<style scoped>
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
