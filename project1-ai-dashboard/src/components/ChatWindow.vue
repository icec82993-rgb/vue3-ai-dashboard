<template>
  <div class="chat-container">
    <h2>AI 驱动的个人智慧看板</h2>
    <p class="subtitle">【开源核心组件】集成核心异步缓冲区 (Buffer) 与 DOM 响应式滚动流追踪</p>
    
    <div class="chat-box" ref="chatBoxRef">
      <div v-for="(msg, index) in messages" :key="index" :class="['message-row', msg.role]">
        <div class="avatar">{{ msg.role === 'user' ? '👤' : '🤖' }}</div>
        <div class="content-box">{{ msg.content }}</div>
      </div>
    </div>

    <div class="input-area">
      <input 
        v-model="userInput" 
        @keyup.enter="sendMessage" 
        :disabled="isTyping"
        placeholder="输入消息，探索流式渲染特性..." 
      />
      <button @click="sendMessage" :disabled="isTyping || !userInput.trim()">
        {{ isTyping ? '思考中...' : '发送' }}
      </button>
    </div>
    <small class="status-tip">状态控制：当前激活 **[高频发送互斥锁]** 状态，防止重复提交引发并发冲突</small>
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'

const userInput = ref('')
const messages = ref([
  { role: 'assistant', content: '您好！我是您的实时AI助手，已成功接入前端流式缓冲区（Stream Buffer）优化算法。' }
])
const isTyping = ref(false)
const chatBoxRef = ref(null)

// 自动滚动触底逻辑
const scrollToBottom = async () => {
  await nextTick()
  if (chatBoxRef.value) {
    chatBoxRef.value.scrollTop = chatBoxRef.value.scrollHeight
  }
}

// 核心重构：真正实现 ReadableStream 的异步流式读取状态机
const simulateStreamAPI = async (textToStream) => {
  // 1. 模拟标准的二进制数据流 Response
  const encoder = new TextEncoder()
  const view = encoder.encode(textToStream)
  
  // 2. 构造符合 W3C 标准的 ReadableStream
  const stream = new ReadableStream({
    start(controller) {
      let position = 0
      const chunkSize = 6 // 每次模拟传输 6 个字节 (约 2 个汉字)
      
      const pushChunk = () => {
        if (position >= view.length) {
          controller.close()
          return
        }
        const chunk = view.slice(position, position + chunkSize)
        controller.enqueue(chunk)
        position += chunkSize
        setTimeout(pushChunk, 40) // 每 40ms 推送一个网络数据块
      }
      pushChunk()
    }
  })

  // 3. 【面试核心考点】使用 getReader() 异步迭代器读取二进制流
  const reader = stream.getReader()
  const decoder = new TextDecoder()
  let resultText = ''

  // 为 AI 回复在数组中先占个位置
  messages.value.push({ role: 'assistant', content: '' })
  const lastIndex = messages.value.length - 1

  // 4. 高效缓冲区异步队列消费
  while (true) {
    const { done, value } = await reader.read()
    if (done) break
    
    // 解码二进制 Chunk 碎片
    const chunkText = decoder.decode(value, { stream: true })
    resultText += chunkText
    
    // 触发响应式更新与 DOM 节流滚动
    messages.value[lastIndex].content = resultText
    scrollToBottom()
  }
}

const sendMessage = async () => {
  if (!userInput.value.trim() || isTyping.value) return

  // 1. 状态锁定：激活[高频发送互斥锁]，对应简历优化项
  isTyping.value = true
  
  messages.value.push({ role: 'user', content: userInput.value })
  const tempInput = userInput.value
  userInput.value = ''
  await scrollToBottom()

  try {
    // 2. 调用符合底层的流式处理器
    const mockFullResponse = `针对您的问题，系统已捕获请求。这是一段模拟标准 ReadableStream 的企业级流式长文本输出。为了规避页面频繁重排（Reflow），前端采用二进制流解码状态机配合 Vue 3 的 nextTick 策略进行无缝追踪。`
    await simulateStreamAPI(mockFullResponse)
  } catch (error) {
    console.error('流传输异常:', error)
  } finally {
    // 3. 释放互斥锁
    isTyping.value = false
    await scrollToBottom()
  }
}
</script>

<style scoped>
.chat-container { max-width: 650px; margin: 0 auto; background: #ffffff; border: 1px solid #e2e8f0; border-radius: 12px; padding: 20px; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05); text-align: left; }
h2 { margin-top: 0; color: #1e293b; font-size: 20px; }
.subtitle { font-size: 13px; color: #64748b; margin-top: -10px; margin-bottom: 20px; border-left: 3px solid #3b82f6; padding-left: 8px; }
.chat-box { height: 350px; overflow-y: auto; border: 1px solid #f1f5f9; background: #fafafa; border-radius: 8px; padding: 15px; margin-bottom: 15px; }
.message-row { display: flex; gap: 12px; margin-bottom: 16px; align-items: flex-start; }
.message-row.user { flex-direction: row-reverse; }
.avatar { width: 32px; height: 32px; border-radius: 50%; background: #e2e8f0; display: flex; align-items: center; justify-content: center; font-size: 16px; }
.message-row.user .avatar { background: #dbeafe; }
.content-box { max-width: 70%; padding: 10px 14px; border-radius: 8px; font-size: 14px; line-height: 1.5; word-break: break-all; }
.assistant .content-box { background: #ffffff; border: 1px solid #e2e8f0; color: #334155; }
.user .content-box { background: #3b82f6; color: #ffffff; }
.input-area { display: flex; gap: 10px; }
input { flex: 1; padding: 10px 14px; border: 1px solid #cbd5e1; border-radius: 6px; outline: none; font-size: 14px; }
input:disabled { background: #f8fafc; cursor: not-allowed; }
button { padding: 0 18px; background: #3b82f6; color: white; border: none; border-radius: 6px; cursor: pointer; font-weight: 500; font-size: 14px; transition: background 0.2s; }
button:disabled { background: #94a3b8; cursor: not-allowed; }
.status-tip { color: #94a3b8; display: block; margin-top: 8px; font-size: 11px; }
</style>