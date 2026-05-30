<template>
  <div class="dashboard-app">
    <h1>AI 驱动的个人智慧看板</h1>
    <hr />
    <ChatWindow />
  </div>
</template>

<script setup>
// 引入刚刚新建的组件
import ChatWindow from './components/ChatWindow.vue'
</script>

<style>
.dashboard-app {
  font-family: Arial, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}
hr {
  margin: 20px 0;
  border: 0;
  border-top: 1px solid #eee;
}
</style>