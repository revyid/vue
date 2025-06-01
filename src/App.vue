<template>
  <div id="app" class="min-h-screen bg-black text-white overflow-x-hidden">
    <!-- Floating Navbar -->
    <FloatingNavbar />

    <!-- Main Content -->
    <main class="relative">
      <HeroSection />
      <ProjectsSection />
      <SkillsSection />
      <ContactSection />
    </main>

    <!-- Footer -->
    <AppFooter />
  </div>
</template>

<script setup lang="ts">
import { onMounted } from 'vue'
import FloatingNavbar from './components/FloatingNavbar.vue'
import HeroSection from './components/HeroSection.vue'
import ProjectsSection from './components/ProjectsSection.vue'
import SkillsSection from './components/SkillsSection.vue'
import ContactSection from './components/ContactSection.vue'
import AppFooter from './components/AppFooter.vue'

onMounted(() => {
  // Initialize AOS or any other animations
  if (typeof window !== 'undefined') {
    // Add any initialization code here
  }
})
</script>

<style>
@import 'tailwindcss';

/* Custom scrollbar */
::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
}

::-webkit-scrollbar-thumb {
  background: #42b883;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
  background: #369870;
}

/* Smooth scrolling */
html {
  scroll-behavior: smooth;
}

/* Glassmorphism utility */
.glass {
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

/* Animation utilities */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeInLeft {
  from {
    opacity: 0;
    transform: translateX(-30px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes fadeInRight {
  from {
    opacity: 0;
    transform: translateX(30px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.animate-fade-in-up {
  animation: fadeInUp 0.6s ease-out forwards;
}

.animate-fade-in-left {
  animation: fadeInLeft 0.6s ease-out forwards;
}

.animate-fade-in-right {
  animation: fadeInRight 0.6s ease-out forwards;
}

/* Delayed animations */
.animate-delay-100 {
  animation-delay: 0.1s;
}
.animate-delay-200 {
  animation-delay: 0.2s;
}
.animate-delay-300 {
  animation-delay: 0.3s;
}
.animate-delay-400 {
  animation-delay: 0.4s;
}
.animate-delay-500 {
  animation-delay: 0.5s;
}
</style>
