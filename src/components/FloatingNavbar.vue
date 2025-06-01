<template>
  <!-- Enhanced Navigation with AI Chat Toggle -->
  <nav 
    class="fixed top-6 left-1/2 transform -translate-x-1/2 z-50 transition-all duration-500 ease-out"
    :class="{ 
      'translate-y-0 opacity-100': isNavVisible, 
      '-translate-y-20 opacity-0': !isNavVisible 
    }"
  >
    <div class="flex items-center space-x-4">
      <!-- Main Navigation -->
      <div class="glass-enhanced rounded-full px-6 md:px-8 py-3 md:py-4 shadow-2xl backdrop-blur-xl border border-white/20">
        <ul class="flex items-center space-x-4 md:space-x-8 text-xs md:text-sm font-semibold">
          <li v-for="item in navItems" :key="item.id">
            <a 
              :href="`#${item.id}`" 
              @click="scrollTo(item.id)"
              class="relative text-white/80 hover:text-[#42b883] transition-all duration-300 hover:scale-110 transform group whitespace-nowrap"
            >
              {{ item.label }}
              <span class="absolute -bottom-1 left-0 w-0 h-0.5 bg-gradient-to-r from-[#42b883] to-[#35495e] transition-all duration-300 group-hover:w-full"></span>
            </a>
          </li>
        </ul>
      </div>

      <!-- AI Chat Toggle Button -->
      <button
        @click="toggleChat"
        class="glass-enhanced w-12 h-12 md:w-14 md:h-14 rounded-full shadow-2xl hover:shadow-[#42b883]/25 transition-all duration-300 hover:scale-110 transform flex items-center justify-center group border border-white/20"
        :class="{ 'bg-gradient-to-r from-[#42b883] to-[#35495e]': isChatOpen, 'hover:bg-white/10': !isChatOpen }"
        title="AI Assistant"
      >
        <svg 
          v-if="!isChatOpen" 
          class="w-6 h-6 md:w-7 md:h-7 text-white transition-transform duration-300 group-hover:scale-110" 
          fill="none" 
          stroke="currentColor" 
          viewBox="0 0 24 24"
        >
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"></path>
        </svg>
        <svg 
          v-else 
          class="w-6 h-6 md:w-7 md:h-7 text-white transition-transform duration-300" 
          fill="none" 
          stroke="currentColor" 
          viewBox="0 0 24 24"
        >
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
        </svg>
      </button>
    </div>
  </nav>

  <!-- AI Chat Interface - Responsive -->
  <teleport to="body">
    <transition name="chat-slide">
      <div 
        v-if="isChatOpen"
        class="fixed inset-0 z-40 lg:inset-auto lg:top-24 lg:right-8 lg:bottom-8 lg:left-auto"
        :class="chatContainerClasses"
      >
        <!-- Mobile backdrop -->
        <div 
          v-if="isMobile"
          class="absolute inset-0 bg-black/50 backdrop-blur-sm lg:hidden"
          @click="toggleChat"
        ></div>
        
        <!-- Chat Container -->
        <div class="relative bg-black/95 lg:bg-black/90 backdrop-blur-2xl rounded-none lg:rounded-2xl shadow-2xl border-0 lg:border lg:border-white/10 flex flex-col h-full overflow-hidden">
          <!-- Chat Header -->
          <div class="bg-gradient-to-r from-[#42b883] to-[#35495e] p-4 lg:p-4 flex items-center justify-between shrink-0">
            <div class="flex items-center space-x-3">
              <div class="w-3 h-3 bg-green-400 rounded-full animate-pulse"></div>
              <h3 class="text-white font-semibold text-sm lg:text-base">AI Assistant</h3>
              <span class="text-white/60 text-xs hidden md:inline">Powered by OpenRouter</span>
            </div>
            <div class="flex items-center space-x-2">
              <button 
                @click="clearChat"
                class="text-white/70 hover:text-white transition-colors p-2 rounded-lg hover:bg-white/10"
                title="Clear Chat"
              >
                <svg class="w-4 h-4 lg:w-5 lg:h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path>
                </svg>
              </button>
              <button 
                v-if="isMobile"
                @click="toggleChat"
                class="text-white/70 hover:text-white transition-colors p-2 rounded-lg hover:bg-white/10 lg:hidden"
                title="Close Chat"
              >
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
                </svg>
              </button>
            </div>
          </div>

          <!-- Chat Messages Container -->
          <div 
            ref="chatContainer"
            class="flex-1 overflow-y-auto p-3 lg:p-4 space-y-3 lg:space-y-4 scrollbar-thin scrollbar-thumb-white/20 scrollbar-track-transparent min-h-0"
          >
            <div 
              v-for="(message, index) in chatHistory" 
              :key="index"
              class="message-container"
              :class="message.role === 'user' ? 'justify-end' : 'justify-start'"
            >
              <div 
                class="max-w-[85%] lg:max-w-[80%] p-3 rounded-2xl relative group"
                :class="message.role === 'user' 
                  ? 'bg-gradient-to-r from-[#42b883] to-[#35495e] text-white ml-auto' 
                  : 'bg-white/10 backdrop-blur-sm text-white'"
              >
                <!-- Message Content -->
                <div 
                  v-if="message.role === 'assistant' && message.isTyping"
                  class="typing-animation text-sm lg:text-base"
                  v-html="message.displayContent"
                ></div>
                <div 
                  v-else
                  class="message-content text-sm lg:text-base"
                  v-html="formatMessage(message.content)"
                ></div>
                
                <!-- Message Actions -->
                <div class="absolute top-2 right-2 opacity-0 group-hover:opacity-100 transition-opacity duration-200">
                  <button 
                    v-if="message.role === 'assistant'"
                    @click="copyMessage(message.content)"
                    class="text-white/60 hover:text-white transition-colors p-1 rounded"
                    title="Copy message"
                  >
                    <svg class="w-3 h-3 lg:w-4 lg:h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"></path>
                    </svg>
                  </button>
                </div>
              </div>
            </div>

            <!-- Typing Indicator -->
            <div v-if="isAiTyping" class="flex justify-start">
              <div class="bg-white/10 backdrop-blur-sm p-3 rounded-2xl">
                <div class="flex space-x-1">
                  <div class="w-2 h-2 bg-[#42b883] rounded-full animate-bounce"></div>
                  <div class="w-2 h-2 bg-[#42b883] rounded-full animate-bounce" style="animation-delay: 0.1s"></div>
                  <div class="w-2 h-2 bg-[#42b883] rounded-full animate-bounce" style="animation-delay: 0.2s"></div>
                </div>
              </div>
            </div>
          </div>

          <!-- Chat Input -->
          <div class="p-3 lg:p-4 border-t border-white/10 shrink-0">
            <div class="flex space-x-2">
              <input
                v-model="currentMessage"
                @keypress.enter="sendMessage"
                placeholder="Ask me anything..."
                class="flex-1 bg-white/10 backdrop-blur-sm border border-white/20 rounded-xl px-3 lg:px-4 py-2 lg:py-2.5 text-sm lg:text-base text-white placeholder-white/50 focus:outline-none focus:border-[#42b883] transition-colors"
                :disabled="isAiTyping"
              />
              <button
                @click="sendMessage"
                :disabled="!currentMessage.trim() || isAiTyping"
                class="bg-gradient-to-r from-[#42b883] to-[#35495e] text-white px-3 lg:px-4 py-2 lg:py-2.5 rounded-xl hover:shadow-lg transition-all duration-300 disabled:opacity-50 disabled:cursor-not-allowed shrink-0"
              >
                <svg class="w-4 h-4 lg:w-5 lg:h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8"></path>
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </teleport>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick, watch, computed } from 'vue'

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

// Responsive state
const isMobile = ref(false)

// Computed properties
const chatContainerClasses = computed(() => ({
  'w-full h-full': isMobile.value,
  'w-96 h-[600px]': !isMobile.value
}))

// Environment variables - Replace with your actual API key
const OPENROUTER_API_KEY = import.meta.env.VITE_OPENROUTER_API_KEY
const OPENROUTER_API_URL = 'https://openrouter.ai/api/v1/chat/completions'

// System prompt for AI personality
const SYSTEM_PROMPT = `You are a sophisticated AI assistant integrated into a modern developer's portfolio website. Your core responsibilities include:

**Portfolio Expertise:**
- Detailed knowledge of web development technologies (Vue.js, React, JavaScript, TypeScript, CSS, HTML)
- Understanding of modern development practices, tools, and frameworks
- Ability to explain complex technical concepts in accessible terms

**Professional Assistance:**
- Help visitors understand the developer's skills, projects, and technical expertise
- Provide insights on web development best practices and industry trends
- Offer guidance on project architecture, implementation strategies, and code optimization
- Assist with debugging, code reviews, and technical problem-solving

**Communication Style:**
- Professional yet approachable and conversational
- Provide detailed, well-structured responses with practical examples
- Use markdown formatting for better readability (code blocks, lists, emphasis)
- Be encouraging and supportive while maintaining technical accuracy

**Key Guidelines:**
- Always provide working code examples when relevant
- Reference modern web development practices and current industry standards
- Explain the "why" behind technical decisions, not just the "how"
- Offer multiple solutions when appropriate, explaining trade-offs
- Stay current with emerging technologies and development trends

**Context Awareness:**
You're currently integrated into a Vue.js portfolio featuring modern responsive design, glassmorphism UI elements, and advanced interactive features. The site demonstrates expertise in modern web technologies and user experience design.

Remember: You're not just answering questions - you're representing professional development expertise and helping visitors understand the quality and depth of technical knowledge showcased in this portfolio.`

let lastScrollY = 0
let scrollTimeoutId: number | null = null

// Optimized navigation scroll handler
const handleScroll = () => {
  if (scrollTimeoutId) {
    cancelAnimationFrame(scrollTimeoutId)
  }
  
  scrollTimeoutId = requestAnimationFrame(() => {
    const currentScrollY = window.scrollY
    const scrollDelta = Math.abs(currentScrollY - lastScrollY)
    
    // Only update if significant scroll change (performance optimization)
    if (scrollDelta > 5) {
      isNavVisible.value = currentScrollY < lastScrollY || currentScrollY < 100
      lastScrollY = currentScrollY
    }
  })
}

// Responsive detection
const updateResponsiveState = () => {
  isMobile.value = window.innerWidth < 1024
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
    addAssistantMessage("👋 Hello! I'm your AI assistant. I can help you with web development questions, explain the projects showcased here, discuss modern development practices, or assist with any coding challenges you're facing. What would you like to explore today?")
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
  } else {
    message.displayContent = content
  }
  
  saveChatHistory()
  scrollToBottom()
}

const typeMessage = async (message: any) => {
  const text = message.content
  let currentIndex = 0
  
  const typeInterval = setInterval(() => {
    if (currentIndex < text.length) {
      message.displayContent = text.substring(0, currentIndex + 1)
      currentIndex++
      scrollToBottom()
    } else {
      message.isTyping = false
      clearInterval(typeInterval)
    }
  }, 30) // Faster typing for better UX
}

const sendMessage = async () => {
  if (!currentMessage.value.trim() || isAiTyping.value) return

  // Check if API key is configured
  if (!OPENROUTER_API_KEY) {
    addAssistantMessage("⚠️ OpenRouter API key is not configured. Please add VITE_OPENROUTER_API_KEY to your .env file.", false)
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
        max_tokens: 1500,
        stream: false
      })
    })

    if (!response.ok) {
      const errorData = await response.text()
      throw new Error(`API Error (${response.status}): ${errorData}`)
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
        errorMessage += "Rate limit reached. Please try again in a moment."
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
  // Enhanced markdown-like formatting
  return content
    .replace(/\*\*(.*?)\*\*/g, '<strong class="font-semibold">$1</strong>')
    .replace(/\*(.*?)\*/g, '<em class="italic">$1</em>')
    .replace(/`(.*?)`/g, '<code class="bg-white/20 px-1.5 py-0.5 rounded text-sm font-mono">$1</code>')
    .replace(/```(\w+)?\n?([\s\S]*?)```/g, '<pre class="bg-white/10 p-3 rounded-lg mt-2 mb-2 overflow-x-auto border border-white/10"><code class="text-sm font-mono">$2</code></pre>')
    .replace(/^### (.*$)/gm, '<h3 class="text-lg font-semibold mt-3 mb-2 text-[#42b883]">$1</h3>')
    .replace(/^## (.*$)/gm, '<h2 class="text-xl font-bold mt-4 mb-2 text-[#42b883]">$1</h2>')
    .replace(/^# (.*$)/gm, '<h1 class="text-2xl font-bold mt-4 mb-3 text-[#42b883]">$1</h1>')
    .replace(/^\- (.*$)/gm, '<li class="ml-4 list-disc">$1</li>')
    .replace(/^\d+\. (.*$)/gm, '<li class="ml-4 list-decimal">$1</li>')
    .replace(/\n/g, '<br>')
}

const copyMessage = async (content: string) => {
  try {
    await navigator.clipboard.writeText(content)
    // Could add a toast notification here
  } catch (err) {
    console.error('Failed to copy text: ', err)
    // Fallback for older browsers
    const textArea = document.createElement('textarea')
    textArea.value = content
    document.body.appendChild(textArea)
    textArea.select()
    document.execCommand('copy')
    document.body.removeChild(textArea)
  }
}

const clearChat = () => {
  chatHistory.value = []
  localStorage.removeItem('portfolio-chat-history')
  addAssistantMessage("🔄 Chat cleared! Ready for a fresh conversation. How can I assist you today?")
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
  try {
    const saved = localStorage.getItem('portfolio-chat-history')
    if (saved) {
      const history = JSON.parse(saved)
      chatHistory.value = history.map((msg: any) => ({
        ...msg,
        isTyping: false,
        displayContent: msg.content
      }))
    }
  } catch (error) {
    console.error('Error loading chat history:', error)
  }
}

const scrollToBottom = () => {
  nextTick(() => {
    if (chatContainer.value) {
      const container = chatContainer.value
      container.scrollTop = container.scrollHeight
    }
  })
}

// Lifecycle hooks
onMounted(() => {
  window.addEventListener('scroll', handleScroll, { passive: true })
  window.addEventListener('resize', updateResponsiveState, { passive: true })
  updateResponsiveState()
  loadChatHistory()
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
  window.removeEventListener('resize', updateResponsiveState)
  if (scrollTimeoutId) {
    cancelAnimationFrame(scrollTimeoutId)
  }
})

// Watch for chat container changes to auto-scroll
watch(() => chatHistory.value.length, () => {
  scrollToBottom()
}, { flush: 'post' })
</script>

<style scoped>
.glass-enhanced {
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.15);
}

.message-container {
  display: flex;
  align-items: flex-end;
}

.message-content,
.typing-animation {
  word-wrap: break-word;
  line-height: 1.6;
}

.message-content code,
.typing-animation code {
  font-family: 'JetBrains Mono', 'Fira Code', 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
}

.message-content pre,
.typing-animation pre {
  font-family: 'JetBrains Mono', 'Fira Code', 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 0.875rem;
  line-height: 1.5;
}

.message-content h1,
.message-content h2,
.message-content h3 {
  margin-top: 1rem;
  margin-bottom: 0.5rem;
}

.message-content li {
  margin-bottom: 0.25rem;
}

.chat-slide-enter-active,
.chat-slide-leave-active {
  transition: all 0.4s cubic-bezier(0.25, 0.8, 0.25, 1);
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
  scrollbar-color: rgba(255, 255, 255, 0.2) transparent;
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

/* Performance optimizations */
.glass-enhanced,
.message-container,
button {
  will-change: transform;
  transform: translateZ(0);
}

/* Mobile optimizations */
@media (max-width: 1023px) {
  .glass-enhanced {
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
  }
}

/* Smooth transitions for all interactive elements */
* {
  transition-property: transform, opacity, color, background-color, border-color, box-shadow;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

/* Prevent layout shift */
nav {
  contain: layout style paint;
}
</style>