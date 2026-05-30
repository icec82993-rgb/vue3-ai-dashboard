# RAG Knowledge-Base & Prompt Orchestration System (基于知识库的智能检索与 Prompt 编排系统)

本项目是一款基于 **Vue 3 (Composition API) + Vite** 开发的 RAG (检索增强生成) 前端可视化调试看板。项目核心攻克了企业级私有知识库在对接大模型时，上下文组装逻辑不透明、提示词工程（Prompt Engineering）难以调试的痛点。

## 🚀 核心技术亮点与优化实现

### 1. 局部文档切片 (Chunks) 检索与相关度评分机制
- **实现方案**：前端模拟高性能向量数据库（Vector DB）的关键词度量算子。当用户输入提问时，系统在毫秒级内完成私有语料库的检索匹配，动态输出带相似度权重得分（Score）的文档切片队列。

### 2. 动态拦截式 Prompt 上下文组装状态机
- **核心逻辑**：利用 Vue 3 的 **Computed (计算属性)** 构建响应式编排管道。自动将“系统级角色设定（System Role）”、“拦截到的 Chunks 小抄”以及“用户原生问题”进行无缝拼接。
- **面试价值**：向面试官证明自己深刻理解 RAG 系统的底层机理（检索、增强、生成三阶段），具备将原始业务数据转化为结构化 Prompt 的工程能力。

## 📂 项目启动指南

```bash
# 进入项目二目录
cd project2-rag-knowledgebase

# 安装依赖
npm install

# 本地热启动
npm run dev