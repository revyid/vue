<template>
  <!-- Enhanced Navigation -->
  <nav 
    class="fixed top-4 left-1/2 transform -translate-x-1/2 z-50 transition-all duration-500 ease-out px-4"
    :class="{ 
      'translate-y-0 opacity-100': isNavVisible, 
      '-translate-y-20 opacity-0': !isNavVisible 
    }"
  >
    <div class="glass-enhanced rounded-full px-4 sm:px-8 py-3 sm:py-4 shadow-2xl backdrop-blur-xl border border-white/20">
      <ul class="flex items-center space-x-4 sm:space-x-10 text-xs sm:text-sm font-semibold">
        <li v-for="item in navItems" :key="item.id">
          <a 
            :href="`#${item.id}`" 
            @click="scrollTo(item.id)"
            class="relative text-white/80 hover:text-[#42b883] transition-all duration-300 hover:scale-110 transform group px-1 sm:px-0"
          >
            {{ item.label }}
            <span class="absolute -bottom-1 left-0 w-0 h-0.5 bg-gradient-to-r from-[#42b883] to-[#35495e] transition-all duration-300 group-hover:w-full"></span>
          </a>
        </li>
      </ul>
    </div>
  </nav>

  <!-- AI Chat Toggle Button - Positioned next to navigation -->
  <button
    @click="toggleChat"
    class="fixed top-4 right-4 sm:right-8 z-50 w-12 h-12 sm:w-14 sm:h-14 rounded-full glass-enhanced shadow-2xl hover:shadow-[#42b883]/25 transition-all duration-300 hover:scale-110 transform flex items-center justify-center group border border-white/20"
    :class="{ 
      'rotate-45': isChatOpen,
      'bg-gradient-to-r from-[#42b883]/20 to-[#35495e]/20': !isChatOpen,
      'bg-gradient-to-r from-[#42b883]/30 to-[#35495e]/30': isChatOpen
    }"
  >
    <!-- AI Icon when closed -->
    <svg 
      v-if="!isChatOpen" 
      class="w-6 h-6 sm:w-7 sm:h-7 text-white transition-transform duration-300 group-hover:scale-110" 
      fill="none" 
      stroke="currentColor" 
      viewBox="0 0 24 24"
    >
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z"></path>
    </svg>
    <!-- Close Icon when open -->
    <svg 
      v-else 
      class="w-6 h-6 sm:w-7 sm:h-7 text-white transition-transform duration-300" 
      fill="none" 
      stroke="currentColor" 
      viewBox="0 0 24 24"
    >
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
    </svg>
    
    <!-- Online indicator -->
    <div class="absolute -top-1 -right-1 w-3 h-3 bg-green-400 rounded-full animate-pulse border-2 border-white/20"></div>
  </button>

  <!-- AI Chat Interface -->
  <transition name="chat-slide">
    <div 
      v-if="isChatOpen"
      class="fixed inset-4 sm:bottom-6 sm:right-6 sm:top-auto sm:left-auto sm:w-96 sm:h-[600px] backdrop-blur-2xl rounded-2xl shadow-2xl border border-white/10 z-40 flex flex-col overflow-hidden glass-enhanced"
    >
      <!-- Chat Header -->
      <div class="bg-gradient-to-r from-[#42b883] to-[#35495e] p-4 flex items-center justify-between rounded-t-2xl">
        <div class="flex items-center space-x-3">
          <div class="relative">
            <div class="w-8 h-8 bg-white/20 rounded-full flex items-center justify-center">
              <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z"></path>
              </svg>
            </div>
            <div class="absolute -bottom-1 -right-1 w-3 h-3 bg-green-400 rounded-full border-2 border-white"></div>
          </div>
          <div>
            <h3 class="text-white font-semibold text-sm sm:text-base">AI Assistant</h3>
            <p class="text-white/70 text-xs">Always here to help</p>
          </div>
        </div>
        <div class="flex items-center space-x-2">
          <button 
            @click="clearChat"
            class="text-white/70 hover:text-white transition-colors p-2 rounded-lg hover:bg-white/10"
            title="Clear Chat"
          >
            <svg class="w-4 h-4 sm:w-5 sm:h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path>
            </svg>
          </button>
          <!-- Mobile close button -->
          <button 
            @click="isChatOpen = false"
            class="sm:hidden text-white/70 hover:text-white transition-colors p-2 rounded-lg hover:bg-white/10"
            title="Close Chat"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
            </svg>
          </button>
        </div>
      </div>

      <!-- Chat Messages Container -->
      <div 
        ref="chatContainer"
        class="flex-1 overflow-y-auto p-3 sm:p-4 space-y-3 sm:space-y-4 scrollbar-thin scrollbar-thumb-white/20 scrollbar-track-transparent"
      >
        <div 
          v-for="(message, index) in chatHistory" 
          :key="index"
          class="message-container"
          :class="message.role === 'user' ? 'justify-end' : 'justify-start'"
        >
          <div 
            class="max-w-[85%] sm:max-w-[80%] p-3 sm:p-4 rounded-2xl relative group shadow-lg"
            :class="message.role === 'user' 
              ? 'bg-gradient-to-r from-[#42b883] to-[#35495e] text-white ml-auto rounded-br-md' 
              : 'bg-white/10 backdrop-blur-sm text-white border border-white/10 rounded-bl-md'"
          >
            <!-- Assistant Avatar -->
            <div 
              v-if="message.role === 'assistant'"
              class="flex items-start space-x-3"
            >
              <div class="w-6 h-6 bg-gradient-to-r from-[#42b883] to-[#35495e] rounded-full flex items-center justify-center flex-shrink-0 mt-1">
                <svg class="w-3 h-3 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z"></path>
                </svg>
              </div>
              <div class="flex-1">
                <!-- Message Content -->
                <div 
                  v-if="message.isTyping"
                  class="typing-animation text-sm sm:text-base"
                  v-html="message.displayContent"
                ></div>
                <div 
                  v-else
                  class="message-content text-sm sm:text-base"
                  v-html="formatMessage(message.content)"
                ></div>
                
                <!-- Timestamp -->
                <div class="text-xs text-white/50 mt-2">
                  {{ formatTime(message.timestamp) }}
                </div>
              </div>
            </div>
            
            <!-- User Message -->
            <div v-else>
              <div 
                class="message-content text-sm sm:text-base"
                v-html="formatMessage(message.content)"
              ></div>
              <div class="text-xs text-white/70 mt-2 text-right">
                {{ formatTime(message.timestamp) }}
              </div>
            </div>
            
            <!-- Message Actions -->
            <div class="absolute top-2 right-2 opacity-0 group-hover:opacity-100 transition-opacity duration-200">
              <button 
                v-if="message.role === 'assistant'"
                @click="copyMessage(message.content)"
                class="text-white/60 hover:text-white transition-colors p-1 rounded"
                title="Copy message"
              >
                <svg class="w-3 h-3 sm:w-4 sm:h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"></path>
                </svg>
              </button>
            </div>
          </div>
        </div>

        <!-- Typing Indicator -->
        <div v-if="isAiTyping" class="flex justify-start">
          <div class="bg-white/10 backdrop-blur-sm p-3 sm:p-4 rounded-2xl border border-white/10 rounded-bl-md shadow-lg">
            <div class="flex items-center space-x-2">
              <div class="w-6 h-6 bg-gradient-to-r from-[#42b883] to-[#35495e] rounded-full flex items-center justify-center">
                <svg class="w-3 h-3 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z"></path>
                </svg>
              </div>
              <div class="flex space-x-1">
                <div class="w-2 h-2 bg-[#42b883] rounded-full animate-bounce"></div>
                <div class="w-2 h-2 bg-[#42b883] rounded-full animate-bounce" style="animation-delay: 0.1s"></div>
                <div class="w-2 h-2 bg-[#42b883] rounded-full animate-bounce" style="animation-delay: 0.2s"></div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Chat Input -->
      <div class="p-3 sm:p-4 border-t border-white/10 bg-white/5 rounded-b-2xl">
        <div class="flex space-x-2 sm:space-x-3">
          <div class="flex-1 relative">
            <input
              v-model="currentMessage"
              @keypress.enter="sendMessage"
              placeholder="Ask me anything..."
              class="w-full bg-white/10 backdrop-blur-sm border border-white/20 rounded-xl px-3 sm:px-4 py-2 sm:py-3 text-white placeholder-white/50 focus:outline-none focus:border-[#42b883] focus:ring-2 focus:ring-[#42b883]/20 transition-all text-sm sm:text-base pr-12"
              :disabled="isAiTyping"
            />
            <!-- Send button for mobile (inside input) -->
            <button
              @click="sendMessage"
              :disabled="!currentMessage.trim() || isAiTyping"
              class="absolute right-2 top-1/2 transform -translate-y-1/2 sm:hidden w-8 h-8 bg-gradient-to-r from-[#42b883] to-[#35495e] text-white rounded-lg hover:shadow-lg transition-all duration-300 disabled:opacity-50 disabled:cursor-not-allowed flex items-center justify-center"
            >
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8"></path>
              </svg>
            </button>
          </div>
          <!-- Send button for desktop -->
          <button
            @click="sendMessage"
            :disabled="!currentMessage.trim() || isAiTyping"
            class="hidden sm:flex bg-gradient-to-r from-[#42b883] to-[#35495e] text-white px-4 py-3 rounded-xl hover:shadow-lg transition-all duration-300 disabled:opacity-50 disabled:cursor-not-allowed items-center justify-center min-w-[48px]"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8"></path>
            </svg>
          </button>
        </div>
        
        <!-- Quick Actions -->
        <div class="flex flex-wrap gap-2 mt-3 sm:mt-4">
          <button
            v-for="quickAction in quickActions"
            :key="quickAction.text"
            @click="sendQuickMessage(quickAction.text)"
            class="px-3 py-1 bg-white/10 hover:bg-white/20 text-white/80 hover:text-white rounded-full text-xs transition-all duration-200 border border-white/10 hover:border-white/20"
            :disabled="isAiTyping"
          >
            {{ quickAction.label }}
          </button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick, watch } from 'vue'

// Environment variables
const OPENROUTER_API_KEY = import.meta.env.VITE_OPENROUTER_API_KEY
const OPENROUTER_API_URL = 'https://openrouter.ai/api/v1/chat/completions'

// Navigation state
const isNavVisible = ref(true)
const navItems = ref([
  { id: 'home', label: 'Home' },
  { id: 'projects', label: 'Projects' },
  { id: 'skills', label: 'Skills' },
  { id: 'contact', label: 'Contact' }
])

// Chat state
const isChatOpen = ref(false)
const chatHistory = ref<Array<{
  role: 'user' | 'assistant'
  content: string
  displayContent?: string
  isTyping?: boolean
  timestamp: number
}>>([])
const currentMessage = ref('')
const isAiTyping = ref(false)
const chatContainer = ref<HTMLElement>()

// Quick actions
const quickActions = ref([
  { label: '👋 Hello', text: 'Hello! Can you introduce yourself?' },
  { label: '💼 Projects', text: 'Tell me about the projects in this portfolio' },
  { label: '🛠️ Skills', text: 'What technologies and skills are showcased here?' },
  { label: '📱 Contact', text: 'How can I get in touch?' }
])

// System prompt for AI personality
const SYSTEM_PROMPT = `You are a helpful and knowledgeable AI assistant integrated into a developer's portfolio website. Your role is to:

1. **Portfolio Assistant**: Help visitors understand the developer's skills, projects, and experience
2. **Technical Expert**: Provide detailed explanations about web development, programming concepts, and technologies
3. **Code Helper**: Assist with coding questions, debugging, and best practices
4. **Project Advisor**: Offer insights on project ideas, architecture decisions, and implementation strategies

Key Guidelines:
- Be conversational but professional
- Provide code examples when relevant
- Use markdown formatting for better readability
- Be encouraging and supportive
- Stay up-to-date with modern web development practices
- Reference the portfolio content when appropriate
- Keep responses concise but informative
- Use emojis sparingly but effectively

Current Context: You're chatting with someone viewing a Vue.js developer's portfolio that features modern web technologies, responsive design, and interactive elements.`

let lastScrollY = 0

// Navigation scroll handler
const handleScroll = () => {
  const currentScrollY = window.scrollY
  isNavVisible.value = currentScrollY < lastScrollY || currentScrollY < 100
  lastScrollY = currentScrollY
}

// Smooth scroll to section
const scrollTo = (elementId: string) => {
  const element = document.getElementById(elementId)
  if (element) {
    element.scrollIntoView({ 
      behavior: 'smooth',
      block: 'start'
    })
  }
}

// Chat functionality
const toggleChat = () => {
  isChatOpen.value = !isChatOpen.value
  if (isChatOpen.value && chatHistory.value.length === 0) {
    // Welcome message
    addAssistantMessage("👋 Hello! I'm your AI assistant. I can help you with web development questions, explain projects, or discuss anything tech-related. How can I assist you today?")
  }
}

const addAssistantMessage = (content: string, useTypingEffect = true) => {
  const message = {
    role: 'assistant' as const,
    content,
    displayContent: '',
    isTyping: useTypingEffect,
    timestamp: Date.now()
  }
  
  chatHistory.value.push(message)
  
  if (useTypingEffect) {
    typeMessage(message)
  }
  
  saveChatHistory()
  scrollToBottom()
}

const typeMessage = async (message: any) => {
  const words = message.content.split(' ')
  let currentIndex = 0
  
  const typeInterval = setInterval(() => {
    if (currentIndex < words.length) {
      message.displayContent = words.slice(0, currentIndex + 1).join(' ')
      currentIndex++
      scrollToBottom()
    } else {
      message.isTyping = false
      message.displayContent = message.content
      clearInterval(typeInterval)
    }
  }, 50) // Adjust typing speed here
}

const sendMessage = async () => {
  if (!currentMessage.value.trim() || isAiTyping.value) return

  const userMessage = {
    role: 'user' as const,
    content: currentMessage.value,
    timestamp: Date.now()
  }

  chatHistory.value.push(userMessage)
  const messageToSend = currentMessage.value
  currentMessage.value = ''
  isAiTyping.value = true

  try {
    if (!OPENROUTER_API_KEY) {
      throw new Error('API key not configured')
    }

    const response = await fetch(OPENROUTER_API_URL, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${OPENROUTER_API_KEY}`,
        'Content-Type': 'application/json',
        'HTTP-Referer': window.location.origin,
        'X-Title': 'Portfolio AI Assistant'
      },
      body: JSON.stringify({
        model: 'deepseek/deepseek-chat:free',
        messages: [
          { role: 'system', content: SYSTEM_PROMPT },
          ...chatHistory.value.slice(-10).map(msg => ({
            role: msg.role,
            content: msg.content
          }))
        ],
        temperature: 0.7,
        max_tokens: 1000
      })
    })

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const data = await response.json()
    const aiResponse = data.choices[0]?.message?.content || "Sorry, I couldn't process your request."
    
    addAssistantMessage(aiResponse)
  } catch (error) {
    console.error('Error calling OpenRouter API:', error)
    let errorMessage = "Sorry, I'm experiencing some technical difficulties. Please try again later."
    
    if (error instanceof Error && error.message === 'API key not configured') {
      errorMessage = "API key is not configured. Please check your environment variables."
    }
    
    addAssistantMessage(errorMessage, false)
  } finally {
    isAiTyping.value = false
    saveChatHistory()
  }
}

const sendQuickMessage = (message: string) => {
  if (isAiTyping.value) return
  currentMessage.value = message
  sendMessage()
}

const formatMessage = (content: string) => {
  // Enhanced markdown-like formatting
  return content
    .replace(/\*\*(.*?)\*\*/g, '<strong class="text-[#42b883]">$1</strong>')
    .replace(/\*(.*?)\*/g, '<em>$1</em>')
    .replace(/`(.*?)`/g, '<code class="bg-white/20 px-2 py-1 rounded text-sm font-mono">$1</code>')
    .replace(/```([\s\S]*?)```/g, '<pre class="bg-white/10 p-3 rounded-lg mt-2 mb-2 overflow-x-auto border border-white/10"><code class="font-mono text-sm">$1</code></pre>')
    .replace(/\n/g, '<br>')
}

const formatTime = (timestamp: number) => {
  const date = new Date(timestamp)
  const now = new Date()
  const diffInMinutes = Math.floor((now.getTime() - date.getTime()) / (1000 * 60))
  
  if (diffInMinutes < 1) return 'Just now'
  if (diffInMinutes < 60) return `${diffInMinutes}m ago`
  if (diffInMinutes < 1440) return `${Math.floor(diffInMinutes / 60)}h ago`
  
  return date.toLocaleDateString()
}

const copyMessage = async (content: string) => {
  try {
    await navigator.clipboard.writeText(content)
    // Could add a toast notification here
  } catch (err) {
    console.error('Failed to copy text: ', err)
  }
}

const clearChat = () => {
  chatHistory.value = []
  if (typeof localStorage !== 'undefined') {
    localStorage.removeItem('portfolio-chat-history')
  }
  addAssistantMessage("Chat cleared! How can I help you today?")
}

const saveChatHistory = () => {
  if (typeof localStorage === 'undefined') return
  
  const historyToSave = chatHistory.value.map(msg => ({
    role: msg.role,
    content: msg.content,
    timestamp: msg.timestamp
  }))
  localStorage.setItem('portfolio-chat-history', JSON.stringify(historyToSave))
}

const loadChatHistory = () => {
  if (typeof localStorage === 'undefined') return
  
  const saved = localStorage.getItem('portfolio-chat-history')
  if (saved) {
    try {
      const history = JSON.parse(saved)
      chatHistory.value = history.map((msg: any) => ({
        ...msg,
        isTyping: false,
        displayContent: msg.content
      }))
    } catch (error) {
      console.error('Error loading chat history:', error)
    }
  }
}

const scrollToBottom = () => {
  nextTick(() => {
    if (chatContainer.value) {
      chatContainer.value.scrollTop = chatContainer.value.scrollHeight
    }
  })
}

// Lifecycle hooks
onMounted(() => {
  window.addEventListener('scroll', handleScroll, { passive: true })
  loadChatHistory()
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
})

// Watch for chat container changes to auto-scroll
watch(() => chatHistory.value.length, () => {
  scrollToBottom()
})
</script>

<style scoped>
.glass-enhanced {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.message-container {
  display: flex;
}

.message-content {
  word-wrap: break-word;
  line-height: 1.6;
}

.message-content code {
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
}

.message-content pre {
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 0.875rem;
  line-height: 1.4;
}

.typing-animation {
  word-wrap: break-word;
  line-height: 1.6;
}

.chat-slide-enter-active,
.chat-slide-leave-active {
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.chat-slide-enter-from {
  opacity: 0;
  transform: translateY(30px) scale(0.9);
}

.chat-slide-leave-to {
  opacity: 0;
  transform: translateY(30px) scale(0.9);
}

/* Custom scrollbar */
.scrollbar-thin {
  scrollbar-width: thin;
}

.scrollbar-thin::-webkit-scrollbar {
  width: 6px;
}

.scrollbar-thin::-webkit-scrollbar-track {
  background: transparent;
}

.scrollbar-thin::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 3px;
}

.scrollbar-thin::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.3);
}

/* Smooth animations */
* {
  transition-property: transform, opacity, color, background-color, border-color;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

/* Performance optimizations */
.glass-enhanced,
.message-container,
button {
  will-change: transform;
  transform: translateZ(0);
}

/* Mobile optimizations */
@media (max-width: 640px) {
  .chat-slide-enter-from,
  .chat-slide-leave-to {
    transform: translateY(50px) scale(0.85);
  }
}

/* Enhanced focus states for accessibility */
button:focus-visible,
input:focus-visible {
  outline: 2px solid #42b883;
  outline-offset: 2px;
}

/* Loading animation */
@keyframes bounce {
  0%, 80%, 100% {
    transform: scale(0);
  }
  40% {
    transform: scale(1);
  }
}

/* Pulse animation for online indicator */
@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.1);
    opacity: 0.7;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

/* Hover effects */
.group:hover .group-hover\:scale-110 {
  transform: scale(1.1);
}

/* Glass morphism enhancements */
.glass-enhanced {
  box-shadow: 
    0 8px 32px 0 rgba(31, 38, 135, 0.37),
    inset 0 1px 0 0 rgba(255, 255, 255, 0.2);
}

/* Message bubble animations */
.message-container {
  animation: messageSlide 0.3s ease-out;
}

@keyframes messageSlide {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Gradient text for AI responses */
.ai-gradient-text {
  background: linear-gradient(135deg, #42b883, #35495e);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

/* Enhanced button hover states */
button:not(:disabled):hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

button:not(:disabled):active {
  transform: translateY(0);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

/* Responsive text sizing */
@media (max-width: 640px) {
  .message-content {
    font-size: 0.875rem;
    line-height: 1.5;
  }
  
  .message-content code {
    font-size: 0.8rem;
  }
  
  .message-content pre {
    font-size: 0.8rem;
  }
}

/* Dark mode compatibility */
@media (prefers-color-scheme: dark) {
  .glass-enhanced {
    background: rgba(0, 0, 0, 0.2);
    border-color: rgba(255, 255, 255, 0.1);
  }
}

/* Accessibility improvements */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* High contrast mode support */
@media (prefers-contrast: high) {
  .glass-enhanced {
    background: rgba(0, 0, 0, 0.8);
    border: 2px solid white;
  }
  
  .text-white\/80 {
    color: white;
  }
  
  .text-white\/70 {
    color: white;
  }
}

</style>