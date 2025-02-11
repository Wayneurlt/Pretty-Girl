<template>
  <div
    class="min-h-screen flex items-center justify-center bg-purple-100 font-sans"
  >
    <div class="text-center p-8 max-w-md mx-4">
      <!-- Animated Jumping Image -->
      <img
        src="/assets/images/jump.png"
        alt="Cute Illustration"
        class="mx-auto mb-6 w-32 h-32 rounded-full object-cover animate-jump"
      />

      <h1 class="text-5xl font-bold text-purple-600 mb-6 animate-fade-in">
        Yaaayyyy!!!!!
      </h1>
    </div>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted } from 'vue';
import confetti from 'canvas-confetti';

// Function to get random number within a range
function randomInRange(min, max) {
  return Math.random() * (max - min) + min;
}

// Function to trigger the confetti fireworks
function startFireworks() {
  const defaults = { startVelocity: 30, spread: 360, ticks: 60, zIndex: 0 };
  
  return setInterval(() => {
    const particleCount = 50;
    
    // Confetti from left side
    confetti({
      ...defaults,
      particleCount,
      origin: { x: randomInRange(0.1, 0.3), y: Math.random() - 0.2 },
    });
    
    // Confetti from right side
    confetti({
      ...defaults,
      particleCount,
      origin: { x: randomInRange(0.7, 0.9), y: Math.random() - 0.2 },
    });
  }, 250);
}

// Start fireworks on component mount
let fireworksInterval;
onMounted(() => {
  fireworksInterval = startFireworks();
});

// Clean up interval on component unmount
onUnmounted(() => {
  if (fireworksInterval) {
    clearInterval(fireworksInterval);
  }
});
</script>

<style scoped>
/* Custom animations */
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

@keyframes jump {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-20px);
  }
}

.animate-fade-in {
  animation: fade-in 1s ease-out;
}

.animate-jump {
  animation: jump 0.6s infinite ease-in-out;
}
</style>