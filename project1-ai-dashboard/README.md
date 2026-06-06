# AI 驱动的个人智慧看板与实时 AI 助手系统

一个基于 Vue 3 + Vite 独立开发的前端项目，集成了 AI 实时对话窗口与安全防御模拟面板。

---

## 项目背景

在学习 Vue 3 的过程中，想找一个真实场景来练手响应式和组件化开发。正好对大模型流式输出比较感兴趣，就做了这个项目，主要是想搞清楚"为什么 ChatGPT 的回复是一个字一个字出来的，前端是怎么处理的"。

---

## 功能介绍

- **AI 对话窗口**：输入问题发送后，AI 回复会像打字机一样逐字出现，而不是等待后一次性显示
- **安全防御模拟面板**：右侧展示模拟的安全审计日志和拦截统计数据
- **响应式布局**：宽屏左右分栏，窄屏自动切换为上下排列

---

## 技术实现

**流式输出处理**

这是这个项目里我觉得最有意思的部分。大模型接口开启 `stream: true` 后返回的不是完整 JSON，而是一段一段的 SSE 数据流。前端用 `ReadableStream` + `getReader()` 循环读取，每读到一小块就解码追加到消息内容里，配合 Vue 3 的响应式自动触发 UI 更新，用户就能看到文字逐渐出现的效果。

```js
const reader = response.body.getReader()
const decoder = new TextDecoder()

while (true) {
  const { done, value } = await reader.read()
  if (done) break
  const chunk = decoder.decode(value, { stream: true })
  // 解析 SSE 格式，追加到消息内容
}
```

**请求状态控制**

用一个 `isTyping` 的 ref 做状态锁，AI 回复期间按钮禁用、输入框禁用，防止重复提交。用 `finally` 块释放锁，保证出错时也能恢复正常。

**自动滚动**

每次有新内容追加时调用 `scrollToBottom()`，里面用 `await nextTick()` 等 Vue 把 DOM 更新完，再去读 `scrollHeight` 设置滚动位置，不然拿到的高度是旧的会滚不到底。

---

## 项目结构

```
src/
├── App.vue              # 根组件，左右分栏布局
└── components/
    ├── ChatWindow.vue   # AI 对话窗口，流式输出核心逻辑
    └── SecurityPanel.vue # 安全日志模拟面板
```

---

## 本地运行

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

> 如果需要真实接入大模型，在 `ChatWindow.vue` 的 `sendMessage` 函数里把模拟调用替换成真实的 fetch 请求，并填入自己的 API Key 即可。

---

## 开发过程中遇到的问题

**nextTick 踩坑**：一开始滚动总是差一条消息的高度，后来发现是 Vue 数据更新和 DOM 更新之间有延迟，加了 `await nextTick()` 之后才正常。

**流式数据解码**：中文字符 UTF-8 编码是 3 个字节，网络分包时一个字可能被切成两半，`decoder.decode(value, { stream: true })` 第二个参数就是处理这个问题的，不加的话偶尔会出现乱码。

---

## 后续计划

- [ ] 接入真实大模型 API 替换掉现在的模拟流
- [ ] 支持多轮对话历史记录
- [ ] 安全面板的日志数据改为动态更新

---

**仓库地址**：[github.com/icec82993-rgb/vue3-ai-dashboard](https://github.com/icec82993-rgb/vue3-ai-dashboard)
