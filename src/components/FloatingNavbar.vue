<template>
  <!-- AI Chat Toggle Button - Positioned above navbar -->
  <button
    @click="toggleChat"
    class="fixed top-6 right-6 z-[60] w-12 h-12 sm:w-14 sm:h-14 rounded-full bg-gradient-to-r from-[#42b883] to-[#35495e] shadow-2xl hover:shadow-[#42b883]/25 transition-all duration-300 hover:scale-110 transform flex items-center justify-center group"
    :class="{ 'rotate-45': isChatOpen }"
  >
    <svg 
      v-if="!isChatOpen" 
      class="w-6 h-6 sm:w-7 sm:h-7 text-white transition-transform duration-300 group-hover:scale-110" 
      fill="none" 
      stroke="currentColor" 
      viewBox="0 0 24 24"
    >
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"></path>
    </svg>
    <svg 
      v-else 
      class="w-6 h-6 sm:w-7 sm:h-7 text-white transition-transform duration-300" 
      fill="none" 
      stroke="currentColor" 
      viewBox="0 0 24 24"
    >
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
    </svg>
  </button>

  <!-- Enhanced Navigation -->
  <nav 
    class="fixed top-6 left-1/2 transform -translate-x-1/2 z-50 transition-all duration-500 ease-out px-4"
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
            class="relative text-white/80 hover:text-[#42b883] transition-all duration-300 hover:scale-110 transform group"
          >
            {{ item.label }}
            <span class="absolute -bottom-1 left-0 w-0 h-0.5 bg-gradient-to-r from-[#42b883] to-[#35495e] transition-all duration-300 group-hover:w-full"></span>
          </a>
        </li>
      </ul>
    </div>
  </nav>

  <!-- AI Chat Interface - Responsive -->
  <transition name="chat-slide">
    <div 
      v-if="isChatOpen"
      class="fixed inset-4 sm:bottom-20 sm:right-6 sm:inset-auto sm:w-96 sm:h-[600px] max-w-full bg-black/90 backdrop-blur-2xl rounded-2xl shadow-2xl border border-white/10 z-40 flex flex-col overflow-hidden"
    >
      <!-- Chat Header -->
      <div class="bg-gradient-to-r from-[#42b883] to-[#35495e] p-3 sm:p-4 flex items-center justify-between shrink-0">
        <div class="flex items-center space-x-3">
          <div class="w-2.5 h-2.5 sm:w-3 sm:h-3 bg-green-400 rounded-full animate-pulse"></div>
          <h3 class="text-white font-semibold text-sm sm:text-base">AI Assistant</h3>
        </div>
        <div class="flex items-center space-x-2">
          <!-- Mobile close button -->
          <button 
            @click="toggleChat"
            class="sm:hidden text-white/70 hover:text-white transition-colors p-1 rounded"
            title="Close Chat"
          >
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
            </svg>
          </button>
          <!-- Clear chat button -->
          <button 
            @click="clearChat"
            class="text-white/70 hover:text-white transition-colors p-1 rounded"
            title="Clear Chat"
          >
            <svg class="w-4 h-4 sm:w-5 sm:h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path>
            </svg>
          </button>
        </div>
      </div>

      <!-- Chat Messages Container -->
      <div 
        ref="chatContainer"
        class="flex-1 overflow-y-auto p-3 sm:p-4 space-y-3 sm:space-y-4 scrollbar-thin scrollbar-thumb-white/20 scrollbar-track-transparent min-h-0"
      >
        <div 
          v-for="(message, index) in chatHistory" 
          :key="index"
          class="message-container"
          :class="message.role === 'user' ? 'justify-end' : 'justify-start'"
        >
          <div 
            class="max-w-[90%] sm:max-w-[85%] p-2.5 sm:p-3 rounded-2xl relative group"
            :class="message.role === 'user' 
              ? 'bg-gradient-to-r from-[#42b883] to-[#35495e] text-white ml-auto' 
              : 'bg-white/10 backdrop-blur-sm text-white'"
          >
            <!-- Message Content -->
            <div 
              v-if="message.role === 'assistant' && message.isTyping"
              class="typing-animation text-sm sm:text-base"
              v-html="message.displayContent"
            ></div>
            <div 
              v-else
              class="message-content text-sm sm:text-base"
              v-html="formatMessage(message.content)"
            ></div>
            
            <!-- Message Actions -->
            <div class="absolute top-1 right-1 sm:top-2 sm:right-2 opacity-0 group-hover:opacity-100 transition-opacity duration-200">
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
          <div class="bg-white/10 backdrop-blur-sm p-2.5 sm:p-3 rounded-2xl">
            <div class="flex space-x-1">
              <div class="w-1.5 h-1.5 sm:w-2 sm:h-2 bg-[#42b883] rounded-full animate-bounce"></div>
              <div class="w-1.5 h-1.5 sm:w-2 sm:h-2 bg-[#42b883] rounded-full animate-bounce" style="animation-delay: 0.1s"></div>
              <div class="w-1.5 h-1.5 sm:w-2 sm:h-2 bg-[#42b883] rounded-full animate-bounce" style="animation-delay: 0.2s"></div>
            </div>
          </div>
        </div>
      </div>

      <!-- Chat Input -->
      <div class="p-3 sm:p-4 border-t border-white/10 shrink-0">
        <div class="flex space-x-2">
          <input
            v-model="currentMessage"
            @keypress.enter="sendMessage"
            placeholder="Ask me anything..."
            class="flex-1 bg-white/10 backdrop-blur-sm border border-white/20 rounded-xl px-3 sm:px-4 py-2 text-sm sm:text-base text-white placeholder-white/50 focus:outline-none focus:border-[#42b883] transition-colors"
            :disabled="isAiTyping"
          />
          <button
            @click="sendMessage"
            :disabled="!currentMessage.trim() || isAiTyping"
            class="bg-gradient-to-r from-[#42b883] to-[#35495e] text-white px-3 sm:px-4 py-2 rounded-xl hover:shadow-lg transition-all duration-300 disabled:opacity-50 disabled:cursor-not-allowed shrink-0"
          >
            <svg class="w-4 h-4 sm:w-5 sm:h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8"></path>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick, watch } from 'vue'

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

// OpenRouter configuration - Using environment variables
const OPENROUTER_API_KEY = import.meta.env.VITE_OPENROUTER_API_KEY
const OPENROUTER_API_URL = 'https://openrouter.ai/api/v1/chat/completions'

// System prompt for AI personality
const SYSTEM_PROMPT = `You are a helpful and knowledgeable AI assistant integrated into a developer's portfolio website. Your role is to:

1. **Portfolio Assistant**: Help visitors understand the developer's skills, projects, and experience
2. **Technical Expert**: Provide detailed explanations about web development, programming concepts, and technologies
3. **Code Helper**: Assist with coding questions, debugging, and best practices
4. **Project Advisor**: Offer insights on project ideas, architecture decisions, and implementation strategies

Key Guidelines:
- Be conversational but professional
- Provide code examples when relevant (use markdown formatting)
- Keep responses concise but informative
- Be encouraging and supportive
- Stay up-to-date with modern web development practices
- Reference the portfolio content when appropriate
- For mobile users, keep responses shorter and more digestible

Current Context: You're chatting with someone viewing a Vue.js developer's portfolio that features modern web technologies, responsive design, and interactive elements.

Important: Always format code with proper markdown syntax using backticks for inline code and triple backticks for code blocks.`

let lastScrollY = 0

// Navigation scroll handler with throttling for better performance
let scrollTimeout: number | null = null
const handleScroll = () => {
  if (scrollTimeout) return
  
  scrollTimeout = window.setTimeout(() => {
    const currentScrollY = window.scrollY
    isNavVisible.value = currentScrollY < lastScrollY || currentScrollY < 100
    lastScrollY = currentScrollY
    scrollTimeout = null
  }, 10)
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
    // Welcome message - shorter for mobile
    const welcomeMessage = window.innerWidth < 640 
      ? "👋 Hi! I'm your AI assistant. Ask me about web dev, projects, or anything tech-related!"
      : "👋 Hello! I'm your AI assistant. I can help you with web development questions, explain projects, or discuss anything tech-related. How can I assist you today?"
    
    addAssistantMessage(welcomeMessage)
  }
  
  // Focus input on desktop when chat opens
  if (isChatOpen.value && window.innerWidth >= 640) {
    nextTick(() => {
      const input = document.querySelector('input[placeholder="Ask me anything..."]') as HTMLInputElement
      if (input) input.focus()
    })
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
  
  // Faster typing on mobile for better UX
  const typingSpeed = window.innerWidth < 640 ? 30 : 50
  
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
  }, typingSpeed)
}

const sendMessage = async () => {
  if (!currentMessage.value.trim() || isAiTyping.value) return

  // Check if API key is configured
  if (!OPENROUTER_API_KEY) {
    addAssistantMessage("⚠️ OpenRouter API key not configured. Please add VITE_OPENROUTER_API_KEY to your .env file.", false)
    return
  }

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
        max_tokens: window.innerWidth < 640 ? 500 : 1000 // Shorter responses on mobile
      })
    })

    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}))
      throw new Error(`API Error: ${response.status} - ${errorData.error?.message || 'Unknown error'}`)
    }

    const data = await response.json()
    const aiResponse = data.choices[0]?.message?.content || "Sorry, I couldn't process your request."
    
    addAssistantMessage(aiResponse)
  } catch (error) {
    console.error('Error calling OpenRouter API:', error)
    let errorMessage = "Sorry, I'm experiencing some technical difficulties. "
    
    if (error instanceof Error) {
      if (error.message.includes('401')) {
        errorMessage += "Please check your API key configuration."
      } else if (error.message.includes('429')) {
        errorMessage += "Rate limit exceeded. Please try again later."
      } else {
        errorMessage += "Please try again later."
      }
    }
    
    addAssistantMessage(errorMessage, false)
  } finally {
    isAiTyping.value = false
    saveChatHistory()
  }
}

const formatMessage = (content: string) => {
  // Enhanced markdown-like formatting with better mobile support
  return content
    .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
    .replace(/\*(.*?)\*/g, '<em>$1</em>')
    .replace(/`(.*?)`/g, '<code class="bg-white/20 px-1 py-0.5 rounded text-xs sm:text-sm font-mono">$1</code>')
    .replace(/```([\s\S]*?)```/g, '<pre class="bg-white/10 p-2 sm:p-3 rounded-lg mt-2 overflow-x-auto text-xs sm:text-sm"><code class="font-mono whitespace-pre-wrap">$1</code></pre>')
    .replace(/\n/g, '<br>')
}

const copyMessage = async (content: string) => {
  try {
    await navigator.clipboard.writeText(content)
    // Simple feedback - could enhance with toast
    const button = event?.target as HTMLElement
    if (button) {
      const originalTitle = button.title
      button.title = 'Copied!'
      setTimeout(() => {
        button.title = originalTitle
      }, 1000)
    }
  } catch (err) {
    console.error('Failed to copy text: ', err)
  }
}

const clearChat = () => {
  chatHistory.value = []
  localStorage.removeItem('portfolio-chat-history')
  const welcomeMessage = window.innerWidth < 640 
    ? "Chat cleared! What can I help you with?"
    : "Chat cleared! How can I help you today?"
  addAssistantMessage(welcomeMessage)
}

const saveChatHistory = () => {
  try {
    const historyToSave = chatHistory.value.map(msg => ({
      role: msg.role,
      content: msg.content,
      timestamp: msg.timestamp
    }))
    localStorage.setItem('portfolio-chat-history', JSON.stringify(historyToSave))
  } catch (error) {
    console.error('Error saving chat history:', error)
  }
}

const loadChatHistory = () => {
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

// Handle window resize for responsive behavior
const handleResize = () => {
  // Close chat on mobile orientation change for better UX
  if (window.innerWidth < 640 && isChatOpen.value) {
    // Optional: could close chat on mobile orientation change
  }
}

// Lifecycle hooks
onMounted(() => {
  window.addEventListener('scroll', handleScroll, { passive: true })
  window.addEventListener('resize', handleResize, { passive: true })
  loadChatHistory()
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
  window.removeEventListener('resize', handleResize)
  if (scrollTimeout) {
    window.clearTimeout(scrollTimeout)
  }
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
  line-height: 1.5;
}

.message-content code {
  font-family: 'Monaco', 'Menlo', 'SF Mono', 'Cascadia Code', 'Roboto Mono', 'Courier New', monospace;
}

.message-content pre {
  font-family: 'Monaco', 'Menlo', 'SF Mono', 'Cascadia Code', 'Roboto Mono', 'Courier New', monospace;
  line-height: 1.4;
}

.typing-animation {
  word-wrap: break-word;
  line-height: 1.5;
}

.chat-slide-enter-active,
.chat-slide-leave-active {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.chat-slide-enter-from {
  opacity: 0;
  transform: translateY(20px) scale(0.95);
}

.chat-slide-leave-to {
  opacity: 0;
  transform: translateY(20px) scale(0.95);
}

/* Custom scrollbar */
.scrollbar-thin {
  scrollbar-width: thin;
}

.scrollbar-thin::-webkit-scrollbar {
  width: 4px;
}

@media (min-width: 640px) {
  .scrollbar-thin::-webkit-scrollbar {
    width: 6px;
  }
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

/* Mobile optimizations */
@media (max-width: 639px) {
  .message-content pre {
    font-size: 0.75rem;
  }
  
  .message-content code {
    font-size: 0.75rem;
  }
}

/* Performance optimizations */
.glass-enhanced,
.message-container,
button {
  will-change: transform;
  transform: translateZ(0);
}

/* Prevent text selection on buttons */
button {
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

/* Better touch targets on mobile */
@media (max-width: 639px) {
  button {
    min-height: 44px;
    min-width: 44px;
  }
}

/* Smooth font rendering */
.message-content,
.typing-animation {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>