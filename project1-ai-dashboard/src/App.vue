<template>
  <div class="dashboard-layout">
    <div class="left-wing">
      <ChatWindow />
    </div>
    <div class="right-wing">
      <SecurityPanel />
    </div>
  </div>
</template>

<script setup>
import ChatWindow from './components/ChatWindow.vue'
import SecurityPanel from './components/SecurityPanel.vue'
</script>

<style>
body { background-color: #f1f5f9; margin: 0; font-family: system-ui, sans-serif; }
.dashboard-layout { display: flex; max-width: 1100px; margin: 40px auto; gap: 20px; padding: 0 20px; }
.left-wing { flex: 2; }
.right-wing { flex: 1; }

@media (max-width: 768px) {
  .dashboard-layout { flex-direction: column; margin: 20px auto; }
}
</style>