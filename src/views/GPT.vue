<template>
  <div class="min-h-screen bg-gradient-to-br from-violet-50 via-white to-indigo-50">
    <div class="container mx-auto px-4 py-8">
      <div class="flex space-x-6">
        <!-- 左侧模型选择和设置 -->
        <div class="w-72 space-y-4">
          <!-- 模型选择 -->
          <div class="bg-white/80 backdrop-blur-sm rounded-2xl shadow-sm p-6 border border-indigo-50">
            <h3 class="text-lg font-semibold text-gray-800 mb-4 flex items-center">
              <i class="fas fa-robot text-indigo-500 mr-2"></i>
              模型选择
            </h3>
            <div class="space-y-2">
              <button
                v-for="model in models"
                :key="model.id"
                @click="selectModel(model)"
                class="w-full p-3 rounded-xl text-left transition-all relative overflow-hidden group"
                :class="selectedModel.id === model.id ? 'bg-indigo-500 text-white' : 'hover:bg-indigo-50 text-gray-600'"
              >
                <div class="flex items-center space-x-3">
                  <i :class="[model.icon, 'text-lg']"></i>
                  <span class="font-medium">{{ model.name }}</span>
                </div>
                <!-- 选中状态的动画效果 -->
                <div class="absolute inset-0 bg-gradient-to-r from-indigo-400/20 to-purple-400/20 transform scale-x-0 group-hover:scale-x-100 transition-transform origin-left"></div>
              </button>
            </div>
          </div>

          <!-- 对话设置 -->
          <div class="bg-white/80 backdrop-blur-sm rounded-2xl shadow-sm p-6 border border-indigo-50">
            <h3 class="text-lg font-semibold text-gray-800 mb-4 flex items-center">
              <i class="fas fa-sliders-h text-indigo-500 mr-2"></i>
              对话设置
            </h3>
            <div class="space-y-4">
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2 flex items-center">
                  <i class="fas fa-thermometer-half text-orange-400 mr-2"></i>
                  温度 ({{ temperature }})
                </label>
                <input
                  v-model="temperature"
                  type="range"
                  min="0"
                  max="2"
                  step="0.1"
                  class="w-full accent-indigo-500"
                >
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2 flex items-center">
                  <i class="fas fa-text-height text-green-500 mr-2"></i>
                  最大长度
                </label>
                <select v-model="maxTokens" class="w-full rounded-lg border-gray-200 focus:ring-2 focus:ring-indigo-500">
                  <option value="1000">1000 tokens 🌱</option>
                  <option value="2000">2000 tokens 🌿</option>
                  <option value="4000">4000 tokens 🌳</option>
                </select>
              </div>
            </div>
          </div>

          <!-- 快捷提示语 -->
          <div class="bg-white/80 backdrop-blur-sm rounded-2xl shadow-sm p-6 border border-indigo-50">
            <h3 class="text-lg font-semibold text-gray-800 mb-4 flex items-center">
              <i class="fas fa-magic text-purple-500 mr-2"></i>
              快捷提示语
            </h3>
            <div class="space-y-2">
              <button
                v-for="(prompt, index) in quickPrompts"
                :key="index"
                @click="useQuickPrompt(prompt.text)"
                class="w-full p-3 rounded-xl text-left hover:bg-indigo-50 transition-all text-sm text-gray-600 flex items-center space-x-2 group"
              >
                <i :class="[prompt.icon, 'text-gray-400 group-hover:text-indigo-500 transition-colors']"></i>
                <span>{{ prompt.text }}</span>
              </button>
            </div>
          </div>
        </div>

        <!-- 中间聊天区域 -->
        <div class="flex-1 bg-white/90 backdrop-blur-sm rounded-2xl shadow-lg overflow-hidden border border-indigo-50">
          <!-- 聊天记录区域 -->
          <div class="h-[600px] overflow-y-auto p-6 space-y-4" ref="chatContainer">
            <!-- 欢迎消息 -->
            <div v-if="chatMessages.length === 0" class="text-center text-gray-500 mt-20">
              <div class="text-6xl mb-4">🤖</div>
              <div class="text-xl font-medium mb-2">你好！我是 AI 助手</div>
              <div class="text-sm">有什么我可以帮你的吗？</div>
            </div>

            <div v-for="(message, index) in chatMessages" 
              :key="index"
              :class="[
                'flex',
                message.role === 'user' ? 'justify-end' : 'justify-start'
              ]"
            >
              <div :class="[
                'max-w-[80%] rounded-2xl p-4 relative group animate-fade-in',
                message.role === 'user' 
                  ? 'bg-gradient-to-r from-indigo-500 to-purple-500 text-white'
                  : 'bg-gray-100 text-gray-800'
              ]">
                <div class="whitespace-pre-wrap">{{ message.content }}</div>
                <!-- 消息操作按钮 -->
                <div class="absolute right-2 top-2 opacity-0 group-hover:opacity-100 transition-opacity flex space-x-1">
                  <button 
                    @click="copyMessage(message.content)"
                    class="p-1.5 hover:bg-white/20 rounded-lg transition-colors"
                    title="复制"
                  >
                    <i class="fas fa-copy text-xs"></i>
                  </button>
                </div>
              </div>
            </div>

            <!-- 加载动画优化 -->
            <div v-if="loading" class="flex justify-start">
              <div class="bg-gray-100 rounded-2xl p-4">
                <div class="flex items-center space-x-2">
                  <div class="w-2 h-2 bg-indigo-400 rounded-full animate-bounce"></div>
                  <div class="w-2 h-2 bg-purple-400 rounded-full animate-bounce" style="animation-delay: 0.2s"></div>
                  <div class="w-2 h-2 bg-pink-400 rounded-full animate-bounce" style="animation-delay: 0.4s"></div>
                </div>
              </div>
            </div>
          </div>

          <!-- 输入区域 -->
          <div class="border-t border-gray-100 p-4 bg-white/50">
            <div class="flex space-x-4">
              <div class="flex-1 relative">
                <textarea
                  v-model="prompt"
                  rows="3"
                  class="w-full resize-none border rounded-xl p-4 pr-12 focus:ring-2 focus:ring-indigo-500 focus:border-transparent"
                  placeholder="输入你的问题... 💭"
                  @keydown.enter.prevent="generateText"
                ></textarea>
                <div class="absolute right-3 bottom-3 text-gray-400 text-sm">
                  <i class="fas fa-keyboard"></i>
                </div>
              </div>
              <div class="flex flex-col space-y-2">
                <button
                  @click="generateText"
                  :disabled="loading || !prompt.trim()"
                  class="px-6 py-3 bg-gradient-to-r from-indigo-500 to-purple-500 text-white rounded-xl hover:opacity-90 disabled:opacity-50 disabled:cursor-not-allowed transform hover:scale-105 transition-all"
                >
                  <i class="fas fa-paper-plane"></i>
                </button>
                <button
                  @click="clearChat"
                  class="px-6 py-3 bg-gray-100 text-gray-600 rounded-xl hover:bg-gray-200 transform hover:scale-105 transition-all"
                  title="清空对话"
                >
                  <i class="fas fa-trash"></i>
                </button>
              </div>
            </div>
          </div>
        </div>

        <!-- 右侧对话历史 -->
        <div class="w-72">
          <div class="bg-white/80 backdrop-blur-sm rounded-2xl shadow-sm p-6 border border-indigo-50">
            <h3 class="text-lg font-semibold text-gray-800 mb-4 flex items-center">
              <i class="fas fa-history text-indigo-500 mr-2"></i>
              对话历史
            </h3>
            <div class="space-y-2">
              <div
                v-for="(chat, index) in chatHistory"
                :key="index"
                class="p-3 rounded-xl hover:bg-indigo-50 cursor-pointer transition-all group"
                @click="loadChat(chat)"
              >
                <div class="flex items-center space-x-2">
                  <i class="fas fa-comments text-gray-400 group-hover:text-indigo-500"></i>
                  <div>
                    <div class="text-sm font-medium text-gray-700 truncate">
                      {{ chat.title }}
                    </div>
                    <div class="text-xs text-gray-500">
                      {{ chat.date }}
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick, watch } from '@vue/runtime-core'

interface Message {
  role: string
  content: string
}

interface Model {
  id: string
  name: string
  icon: string
}

interface ChatHistory {
  title: string
  date: string
  messages?: Message[]
}

// 模型配置
const models: Model[] = [
  { id: 'gpt-3.5-turbo', name: 'GPT-3.5', icon: 'fas fa-bolt' },
  { id: 'gpt-4', name: 'GPT-4', icon: 'fas fa-brain' }
]

// 快捷提示语
const quickPrompts = [
  { text: '帮我解释一下这段代码', icon: 'fas fa-code' },
  { text: '这个问题如何优化？', icon: 'fas fa-lightbulb' },
  { text: '给我一些建议', icon: 'fas fa-comment-dots' },
  { text: '用更简单的方式解释', icon: 'fas fa-feather' }
]

const my_key = "sk-T6c5Bwnz67laZdm68f4411Ed73F942438fF9Bf543b34C933"
const prompt = ref<string>("")
const chatMessages = ref<Message[]>([])
const loading = ref<boolean>(false)
const error = ref<string>("")
const chatContainer = ref<HTMLElement | null>(null)
const selectedModel = ref<Model>(models[0])
const temperature = ref<number>(0.7)
const maxTokens = ref<number>(2000)

// 对话历史
const chatHistory = ref<ChatHistory[]>([
  { title: '上一次对话', date: '2024-03-20 15:30' },
  { title: '代码优化讨论', date: '2024-03-19 10:15' }
])

// 选择模型
const selectModel = (model: typeof models[0]) => {
  selectedModel.value = model
}

// 使用快捷提示语
const useQuickPrompt = (quickPrompt: string) => {
  prompt.value = quickPrompt
}

// 复制消息内容
const copyMessage = async (content: string) => {
  try {
    await navigator.clipboard.writeText(content)
    // 可以添加一个复制成功的提示
  } catch (err) {
    console.error('复制失败:', err)
  }
}

// 清空对话
const clearChat = () => {
  if (confirm('确定要清空当前对话吗？')) {
    chatMessages.value = []
  }
}

// 加载历史对话
const loadChat = (chat: typeof chatHistory.value[0]) => {
  // 这里可以实现加载历史对话的逻辑
  console.log('加载对话:', chat)
}

const generateText = async () => {
  if (!prompt.value.trim()) return

  try {
    loading.value = true
    error.value = ""

    chatMessages.value.push({
      role: "user",
      content: prompt.value
    })

    prompt.value = ""
    await scrollToBottom()

    const response = await fetch("https://api.xty.app/v1/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${my_key}`
      },
      body: JSON.stringify({
        model: selectedModel.value.id,
        messages: chatMessages.value.map((msg: Message) => ({
          role: msg.role,
          content: msg.content
        })),
        temperature: Number(temperature.value),
        max_tokens: Number(maxTokens.value)
      })
    })

    if (!response.ok) {
      const errorData = await response.json()
      throw new Error(errorData.message || "API 请求失败")
    }

    const data = await response.json()
    
    if (data.error) {
      throw new Error(data.error.message || "获取回复失败")
    }

    chatMessages.value.push({
      role: "assistant",
      content: data.choices[0].message.content
    })

    await scrollToBottom()

  } catch (e) {
    error.value = e instanceof Error ? e.message : "发生错误，请稍后重试"
    console.error("GPT API 错误:", e)
  } finally {
    loading.value = false
  }
}

// 滚动到底部
const scrollToBottom = async () => {
  await nextTick()
  if (chatContainer.value) {
    chatContainer.value.scrollTop = chatContainer.value.scrollHeight
  }
}

// 监听聊天记录变化，自动滚动
watch(chatMessages, () => {
  scrollToBottom()
}, { deep: true })

onMounted(() => {
  scrollToBottom()
})
</script>

<style scoped>
/* 保持原有样式并添加新的样式 */
.overflow-y-auto {
  scrollbar-width: thin;
  scrollbar-color: #CBD5E0 transparent;
}

.overflow-y-auto::-webkit-scrollbar {
  width: 4px;
}

.overflow-y-auto::-webkit-scrollbar-track {
  background: transparent;
}

.overflow-y-auto::-webkit-scrollbar-thumb {
  background-color: #CBD5E0;
  border-radius: 2px;
}

/* 消息动画 */
@keyframes fade-in-up {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.flex > div {
  animation: fade-in-up 0.3s ease-out;
}

/* 添加悬浮效果 */
.hover\:shadow-md:hover {
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}

/* 添加过渡效果 */
.transition-all {
  transition: all 0.3s ease;
}

/* 添加渐变动画 */
.bg-gradient-to-r {
  background-size: 200% 200%;
  animation: gradient 15s ease infinite;
}

@keyframes gradient {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

/* 添加消息动画 */
.animate-fade-in {
  animation: fadeIn 0.3s ease-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 添加毛玻璃效果 */
.backdrop-blur-sm {
  backdrop-filter: blur(8px);
}
</style> 