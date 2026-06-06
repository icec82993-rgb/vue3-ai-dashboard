<template>
  <div class="chat-container"> //最外层容器div，纯粹是布局用的外壳，Class属于桥梁标签，CSS里给它设置了最大宽度、圆角、阴影，让整个聊天窗口看起来像一张对应。
    <h2>AI 驱动的个人智慧看板</h2>
    <p class="subtitle">【开源核心组件】集成核心异步缓冲区 (Buffer) 与 DOM 响应式滚动流追踪</p>
    
    <div class="chat-box" ref="chatBoxRef"> //ref="chatBoxRef"是关键——给这个div绑定一个模板引用，让JS里能够拿到这个DOM元素的实际对象，之后操控scrollTop实现自动滚动。没有这个ref，JS里就找不到这个div。
      <div v-for="(msg, index) in messages" :key="index" :class="['message-row', msg.role]">
        //v-for通过消息渲染渲染每一条消息，每一条消息渲染一个div，msg是当前那条消息的数据，index是它的下标。:key="index"给 Vu​​e 的虚拟 DOM Diff 算法一个唯一标识，让知道哪条消息变了，避免全量重渲染。:class="['message-row', msg.role]"动态绑定类，用户消息加user类（右对齐），助理消息加assistant类（左对齐）。
        <div class="avatar">{{ msg.role === 'user' ? '👤' : '🤖' }}</div>
        //三元表达式，判断当前这条消息是用户发的还是AI发的，显示不同的表情头像。
        <div class="content-box">{{ msg.content }}</div>  //显示消息的文字内容。{{ }}是 Vue 的插值语法，把msg.content值渲染成文本。流式输出时，这个值会不断追加，用户就会看到文字逐渐出现。

      </div>
    </div>

    <div class="input-area">
      <input 
        v-model="userInput"   
        @keyup.enter="sendMessage" 
        :disabled="isTyping"
        placeholder="输入消息，探索流式渲染特性..." 
      />
      //18行： 结构绑定，输入框的内容和userInput该变量实时同步。用户打字，userInput跟着变；代码把userInput清空，输入框也跟着清空。
      19 监听回车键反馈事件，触发发送消息函数，不用点按钮也能发送。
      20 AI正在回复时禁止输入框，阻止用户在此时乱输入。
      21：输入框为空时显示的提示文字，纯UI。
      <button @click="sendMessage" :disabled="isTyping || !userInput.trim()">
        {{ isTyping ? '思考中...' : '发送' }}  //isTyping ? '思考中...' : '发送' — 按钮文字根据状态动态切换，为用户提供清晰的视觉反馈。
        //@click="sendMessage"— 点击触发发送函数，:disabled="isTyping || !userInput.trim()"— 两个条件任一满足就取消按钮：AI还在回复中，或者输入框是空的（.trim()去掉空格，禁止只输入空格也能发送）。
      </button>
    </div>
    <small class="status-tip">状态控制：当前激活 **[高频发送互斥锁]** 状态，防止重复提交引发并发冲突</small>  //底部的说明文字，静态展示，纯粹是给看代码的人解释用的，没有逻辑。
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'
//从 Vue 完成里只引入这两个函数。ref用于创建响应式数据，nextTick用于等待 DOM 更新。其次引入，不会把整个 Vue 都备用引入。
const userInput = ref('')  //创建一个响应式字符串，初始值为空字符串，和输入框绑定绑定。用ref是因为它是唯一值。
const messages = ref([
  { role: 'assistant', content: '您好！我是您的实时AI助手，已成功接入前端流式缓冲区（Stream Buffer）优化算法。' }
])  //创建消息修改吞吐量，最终得到一条 AI 的欢迎消息。用ref包裹是因为后面要对吞吐量里某些元素的属性做精准（messages.value[lastIndex].content = resultText），Vue 需要通过.value追踪这个变化。
const isTyping = ref(false)  //状态锁定。false表示空闲可以发送，true表示AI正在回复中。这一个变量同时控制了：输入框禁用、按钮禁用、按钮文字切换。
const chatBoxRef = ref(null)  //和模板里ref="chatBoxRef"定位。初始值是null，组件挂载到页面后 Vue 会自动把那个 div 的真实 DOM 对象属性给它。之后chatBoxRef.value可以用拿到那个 div。

// 自动滚动触底逻辑
const scrollToBottom = async () => {  ////async— 函数内部使用await，所以声明为异步函数。
  await nextTick()  //await nextTick()— Vue 数据变化后不会立即更新 DOM，而是把更新放到下一个微任务队列里批量执行。await nextTick()就是“暂停在这里，等 Vue 把 DOM 刷新完，再往下走”。如果不等，下一行读到的scrollHeight是旧值，滚动位置会差一条消息的高度。
  if (chatBoxRef.value) {  //防御性判断，确认 DOM 元素存在才操作，避免空指针报错
    chatBoxRef.value.scrollTop = chatBoxRef.value.scrollHeight
  }
}
//scrollTop = scrollHeight—scrollTop是当前滚动位置，scrollHeight是内容总高度。把滚动位置设置成内容总高度，就等于强制滚动到最底部。
// 核心重构：真正实现 ReadableStream 的异步流式读取状态机
const simulateStreamAPI = async (textToStream) => {  //这个函数接收一段完整文本，把它模拟成像真实的网络流式传输一样，一点一点地“推”出来。参数textToStream就是要流式输出的那段完整文字。
  // 1. 模拟标准的二进制数据流 Response
  const encoder = new TextEncoder()
  const view = encoder.encode(textToStream)
  //TextEncoder把字符串转成Uint8Array（二进制字节数组）。之所以要转成二进制，是因为真实的网络传输走的就是二进制，这里模拟真实的情况。view就是那段文字的二进制形式。
  // 2. 构造符合 W3C 标准的 ReadableStream
  const stream = new ReadableStream({
    start(controller) {  //手动创建一个ReadableStream对象。ReadableStream是浏览器原生的Web API，代表一个可以逐块读取的数据流。start(controller)这个Stream一创建就自动执行的回调函数，controller是控制器，负责往流里推数据和关闭流。
      let position = 0  //let和const区别在前者可以修改，后者不可以
      const chunkSize = 6 // 每次模拟传输 6 个字节 (约 2 个汉字)
      
      const pushChunk = () => {
        if (position >= view.length) {  //if (position >= view.length)— 如果已经读完所有字节。
          controller.close() //，调用controller.close()关闭流，函数结束
          return
        }
        const chunk = view.slice(position, position + chunkSize)//从当前位置切出 6 个字节作为这次要推的数据块。
        controller.enqueue(chunk)  //把这 6 个字节推进流里，下游的阅读器就可以读到了。
        position += chunkSize  //指针往后移 6 位，接下来从这里继续切。
        setTimeout(pushChunk, 40) // 每 40ms 推送一个网络数据块
      }
      pushChunk()//重新执行第一次，然后依靠 setTimeout 下降驱动后续执行
    }
  })

  // 3. 使用 getReader() 异步迭代器读取二进制流
  const reader = stream.getReader()  //锁定这个流并返回一个读取器。一个流同时只能有一个阅读器，锁定后别人就读不了了。
  const decoder = new TextDecoder()  //和前面的TextEncoder反向操作，将二进制字节再转回字符串。
  let resultText = ''  //累积标记，把每次读到的文字片段拼凑起来，形成最终完整的回复。

  // 为 AI 回复在数组中先占个位置
  messages.value.push({ role: 'assistant', content: '' })
  const lastIndex = messages.value.length - 1
//这一行是流式渲染的核心设计：先往消息队列里推一条空的AI消息占位，内容是空字符串。后面每读到一块文字，就更新这条消息的内容，而不是每次都添加一条消息。这样用户看到的效果是：一条AI消息从空白开始，文字逐渐出现。
//lastIndex这个记录的占位消息在数据库里的下标 因为push之后长度加一，减一就是最后一个元素的位置。
  // 4. 高效缓冲区异步队列消费
  while (true) {
    const { done, value } = await reader.read()
    if (done) break
    //这是消费ReadableStream的标准写法。reader.read()每次调用返回一个Promise，解决时给出{ done, value }。done为true表示流关闭了，没有更多数据，跳出循环。value就是这次读到的那块二进制数据。
    // 解码二进制 Chunk 碎片
    const chunkText = decoder.decode(value, { stream: true })  //把这块二进制转成文字。{ stream: true }告诉解码器还没有结束，如果这块数据有不完整的中文字节（一个中文字被忽略了两半），先保留着，等下一块来了再拼完整。
    resultText += chunkText  //追加到累积字符串。
    
    // 触发响应式更新与 DOM 节流滚动
    messages.value[lastIndex].content = resultText  //把累积的文字属性给那条占位消息。Vue检测到这个属性变化，立即触发UI重新渲染，用户就看到文字又多了几个字。
    scrollToBottom()  //有新文字出现，自动滚动到底部，跟随文字走。
  }
}

const sendMessage = async () => {
  if (!userInput.value.trim() || isTyping.value) return  //发送前的双重检查：输入框为空或者AI还在回复中，直接返回什么都不做。.trim()去掉首尾空格，防止只输入空格也能发送。

  // 1. 状态锁定：激活[高频发送互斥锁]，对应简历优化项
  isTyping.value = true  //锁定状态，按钮和输入框取消禁用
  
  messages.value.push({ role: 'user', content: userInput.value })//把输入用户的消息加进列表，UI立即显示用户那条消息。
  const tempInput = userInput.value;//先把输入内容保存下来（虽然这个版本后面没有用到tempInput，但这是一个好习惯，防止清空后还需要用到初始化内容）。
  userInput.value = ''  //清空输入框，用户看到输入框变空了，可以准备下一条消息。
  await scrollToBottom()  // 用户消息添加后滚动到底部。

  try {  //调用流式处理函数，等它运行完成（await保证流读完成才继续）。
    // 2. 调用符合底层的流式处理器
    const mockFullResponse = `针对您的问题，系统已捕获请求。这是一段模拟标准 ReadableStream 的企业级流式长文本输出。为了规避页面频繁重排（Reflow），前端采用二进制流解码状态机配合 Vue 3 的 nextTick 策略进行无缝追踪。`
    await simulateStreamAPI(mockFullResponse)
  } catch (error) {  // 如果流传输过程中抛出异常，捕获错误打印到控制台，防止页面崩溃。
    console.error('流传输异常:', error)
  } finally {  //无论成功还是报错，这里的代码都一定会执行：释放状态锁（isTyping = false），按钮重新可用；滚动到底部确保最新内容可见。finally比把这两行写在try结果更安全，因为报错时try块会中断，但finally不会
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

//max-width: 650px（限制最大宽度不超过px）margin: 0 auto使水平居中；background设置白色背景；border-radius给四个角加圆角，整体外观如一张对应。
//margin-top: 0去掉浏览器默认给h2加的上边距，border加一圈细边框；background设置浅灰背景区分输入区域。
//display: flex开启弹性布局使头像和气泡横向排列；gap: 12px头像和气泡之间留12px距离；margin-bottom: 16px每条消息之间留16px距离；align-items: flex-start头像和气泡顶部对齐。
//flex-direction: row-reverse把排列方向改成从右到左，头像跑到右边，气泡也靠右，实现用户消息右对齐的效果。
//border-radius: 50%把方形变成圆形；background设置头像背景色；display: flex; align-items: center; justify-content: center; font-size: 16px让表情居中显示。
//max-width: 70%限制消息气泡最大宽度，padding: 10px 14px给气泡内文字加内边距；border-radius: 8px圆角；font-size: 14px设置字体大小；line-height: 1.5增加行高让文本更易读；word-break: break-all允许长单词或URL换行，防止气泡被撑破。
.assistant .content-box设置AI消息气泡的背景色、边框和文字颜色