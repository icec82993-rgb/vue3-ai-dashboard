<template>
  <div class="chat-container">
    <div class="messages" ref="msgContainer">
      <div v-for="(msg, index) in messages" :key="index" :class="['msg', msg.role]">
        {{ msg.content }}
      </div>
    </div>
    <div class="input-area">
      <input v-model="userInput" @keyup.enter="sendMessage" placeholder="输入消息..." />
      <button :disabled="isTyping" @click="sendMessage">发送</button>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, nextTick } from 'vue'

const userInput = ref('')
const isTyping = ref(false)
const messages = reactive([
  { role: 'assistant', content: '你好！我是你的实时AI助手，有什么可以帮您？' }
])
const msgContainer = ref(null)

// 滚动到底部，优化高频渲染性能
const scrollToBottom = () => {
  nextTick(() => {
    if (msgContainer.value) {
      msgContainer.value.scrollTop = msgContainer.value.scrollHeight
    }
  })
}

const sendMessage = async () => {
  if (!userInput.value.trim() || isTyping.value) return
  
  const userText = userInput.value
  messages.push({ role: 'user', content: userText })
  userInput.value = ''
  isTyping.value = true
  scrollToBottom()

  // 模拟流式大模型数据（Stream）
  // 实际开发中替换为 fetch('api/chat', { method: 'POST', body: ... })
  const assistantMsg = reactive({ role: 'assistant', content: '' })
  messages.push(assistantMsg)

  const textTemplate = `这是模拟的大模型流式输出长文本...为了防止页面频繁重排（Reflow）导致卡顿，我们结合了分段渲染与 nextTick 节流滚动。确保内容不会发生严重溢出。`
  let index = 0
  
  // 模拟流式定时器
  const interval = setInterval(() => {
    if (index < textTemplate.length) {
      // 每次追加 2 个字符，模拟分段渲染
      assistantMsg.content += textTemplate.slice(index, index + 2)
      index += 2
      scrollToBottom()
    } else {
      clearInterval(interval)
      isTyping.value = false
    }
  }, 50) // 50ms 频率
}
</script>

<style scoped>
.chat-container { display: flex; flex-direction: column; height: 500px; border: 1px solid #ccc; width: 400px; margin: 20px auto; }
.messages { flex: 1; overflow-y: auto; padding: 10px; background: #f9f9f9; }
.msg { margin-bottom: 10px; padding: 8px; border-radius: 4px; max-width: 80%; }
.user { background: #d1e7dd; align-self: flex-end; margin-left: auto; }
.assistant { background: #e2e3e5; }
.input-area { display: flex; border-top: 1px solid #ccc; }
.input-area input { flex: 1; padding: 10px; border: none; }
.input-area button { padding: 10px; }
</style>