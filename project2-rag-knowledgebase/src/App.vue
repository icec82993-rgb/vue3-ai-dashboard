<template>
  <div class="rag-container">
    <h2>基于 Vue 3 的 RAG 知识库检索与 Prompt 调试系统</h2>
    <p class="subtitle">【工程实践】探究知识库局部文档切片（Chunks）与大模型上下文增强的拦截流</p>

    <div class="rag-layout">
      <div class="panel control-panel">
        <h3>1. 输入问题与检索小抄</h3>
        <div class="input-group">
          <input v-model="searchQuery" placeholder="输入你想问的知识库问题...（例如：组件通信）" />
          <button @click="handleRetrieve">向量检索</button>
        </div>

        <div v-if="retrievedChunks.length > 0" class="chunks-section">
          <h4>💡 向量库匹配到的相关文档切片 (Chunks)：</h4>
          <div v-for="(chunk, idx) in retrievedChunks" :key="idx" class="chunk-card">
            <div class="chunk-header">
              <span>📄 切片 #{{ idx + 1 }}</span>
              <span class="score">相关度相似得分: <b>{{ chunk.score }}</b></span>
            </div>
            <p class="chunk-content">{{ chunk.content }}</p>
          </div>
        </div>

        <div v-if="retrievedChunks.length > 0" class="prompt-section">
          <h3>2. 动态组装给大模型的 Prompt 模板</h3>
          <div class="prompt-preview">
            <span class="badge">系统拦截组装完成</span>
            <pre>{{ assembledPrompt }}</pre>
          </div>
          <button class="btn-primary" @click="sendToLLM" :disabled="isGenerating">
            {{ isGenerating ? '大模型正在对照小抄思考...' : '将增强 Prompt 送入大模型' }}
          </button>
        </div>
      </div>

      <div class="panel chat-panel">
        <h3>3. 大模型基于上下文知识的回答</h3>
        <div class="response-box">
          <p v-if="!aiResponse" class="placeholder">暂无输出，请在左侧点击“送入大模型”...</p>
          <p v-else class="ai-text">{{ aiResponse }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const searchQuery = ref('')
const isGenerating = ref(false)
const aiResponse = ref('')
const retrievedChunks = ref([])

// 模拟本地向量数据库的数据
const mockVectorDatabase = [
  { content: 'Vue 3 中父传子使用 defineProps，子传父使用 defineEmits。', keywords: ['组件', '通信', '传参', '父子'] },
  { content: '跨组件全局通信可以使用 Provide / Inject，或者引入 Pinia 状态管理库。', keywords: ['通信', '全局', '跨组件', 'pinia'] },
  { content: '组合式 API (Composition API) 核心依赖 ref 和 reactive 实现响应式数据绑定。', keywords: ['组合式', 'api', 'ref', 'reactive'] }
]

// 核心逻辑 1：模拟局部知识库向量检索
const handleRetrieve = () => {
  if (!searchQuery.value.trim()) return
  
  const query = searchQuery.value.toLowerCase()
  const matches = mockVectorDatabase.filter(item => 
    item.keywords.some(kw => query.includes(kw)) || item.content.includes(query)
  )

  retrievedChunks.value = matches.map(item => ({
    content: item.content,
    score: (0.85 + Math.random() * 0.12).toFixed(4)
  }))

  if (retrievedChunks.value.length === 0) {
    retrievedChunks.value = [{ content: '未在本地知识库匹配到精准切片，大模型将使用自身预训练知识回答。', score: '0.0000' }]
  }
}

// 核心逻辑 2：【面试核心：动态 Prompt 组装计算属性】
const assembledPrompt = computed(() => {
  if (retrievedChunks.value.length === 0) return ''
  const context = retrievedChunks.value.map((c, i) => `[资料${i+1}] ${c.content}`).join('\n')
  return `【系统角色说明】你是一个严谨的知识库助手。请严格基于以下给定的已知信息回答用户问题。
--------
已知参考资料：
${context}
--------
用户真实问题：${searchQuery.value}
--------
请基于上述小抄，进行准确回答：`
})

// 核心逻辑 3：模拟将组装好的高级 Prompt 喂给大模型
const sendToLLM = () => {
  isGenerating.value = true
  aiResponse.value = ''
  
  let fullAnswer = `【基于知识库回答】根据已知资料，如果你想实现组件通信：`
  if (searchQuery.value.includes('通信')) {
    fullAnswer += ` 父子组件建议优先使用 defineProps 和 defineEmits。如果是更复杂的跨层级全局通信，推荐使用 Provide/Inject 或者集成大厂通用的 Pinia 状态树。这可以保证代码的高内聚低耦合。`
  } else {
    fullAnswer += ` 建议确保关键词包含“通信”、“组合式”等，系统会自动在前端检索向量库切片，并合并上下文输入大模型。`
  }

  let i = 0
  const timer = setInterval(() => {
    if (i < fullAnswer.length) {
      aiResponse.value += fullAnswer.charAt(i)
      i++
    } else {
      clearInterval(timer)
      isGenerating.value = false
    }
  }, 30)
}
</script>

<style scoped>
.rag-container { max-width: 1200px; margin: 30px auto; padding: 0 20px; font-family: system-ui, sans-serif; color: #1e293b; text-align: left; }
h2 { font-size: 22px; margin-bottom: 5px; }
.subtitle { font-size: 13px; color: #64748b; margin-top: 0; margin-bottom: 25px; border-left: 3px solid #10b981; padding-left: 8px; }
.rag-layout { display: flex; gap: 20px; }
.panel { background: #ffffff; border: 1px solid #e2e8f0; border-radius: 12px; padding: 20px; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05); }
.control-panel { flex: 1.3; }
.chat-panel { flex: 1; display: flex; flex-direction: column; }
h3 { font-size: 15px; margin-top: 0; color: #334155; border-bottom: 1px solid #f1f5f9; padding-bottom: 8px; }
h4 { font-size: 13px; margin: 15px 0 8px 0; color: #059669; }
.input-group { display: flex; gap: 10px; }
input { flex: 1; padding: 10px 14px; border: 1px solid #cbd5e1; border-radius: 6px; outline: none; font-size: 14px; }
button { padding: 0 16px; background: #10b981; color: white; border: none; border-radius: 6px; cursor: pointer; font-weight: 500; font-size: 14px; }
button:hover { background: #059669; }
.chunk-card { background: #f0fdf4; border: 1px solid #bbf7d0; padding: 12px; border-radius: 6px; margin-bottom: 10px; }
.chunk-header { display: flex; justify-content: space-between; font-size: 11px; color: #166534; margin-bottom: 6px; }
.chunk-content { margin: 0; font-size: 13px; color: #1e293b; line-height: 1.5; }
.prompt-preview { background: #f8fafc; border: 1px solid #e2e8f0; padding: 12px; border-radius: 6px; margin: 12px 0; position: relative; }
pre { margin: 0; font-family: monospace; font-size: 12px; color: #475569; white-space: pre-wrap; word-break: break-all; }
.badge { position: absolute; right: 10px; top: 10px; background: #e2e8f0; font-size: 10px; padding: 2px 6px; border-radius: 4px; color: #64748b; }
.btn-primary { width: 100%; padding: 12px; background: #3b82f6; font-size: 14px; }
.btn-primary:hover { background: #2563eb; }
.btn-primary:disabled { background: #94a3b8; cursor: not-allowed; }
.response-box { flex: 1; background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 8px; padding: 15px; font-size: 14px; line-height: 1.6; }
.placeholder { color: #94a3b8; font-style: italic; text-align: center; margin-top: 40px; }
.ai-text { margin: 0; color: #0f172a; white-space: pre-wrap; }
@media (max-width: 900px) { .rag-layout { flex-direction: column; } }
</style>