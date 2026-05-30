<template>
  <div class="security-container">
    <h3>🛡️ 安全防御实时模拟网关</h3>
    <p class="desc">联动防御状态：<span class="status-tag">运行中</span></p>
    
    <div class="metrics">
      <div class="card">
        <span class="label">拦截恶意攻击</span>
        <span class="value text-red">1,240 次</span>
      </div>
      <div class="card">
        <span class="label">当前安全评分</span>
        <span class="value text-green">98 分</span>
      </div>
    </div>

    <div class="log-box">
      <h4>实时安全审计日志：</h4>
      <ul>
        <li><span class="time">[01:42]</span> SQL 注入攻击威胁已成功拦截 (IP: 192.168.1.105)</li>
        <li><span class="time">[01:38]</span> 敏感词过滤网关拦截成功：涉及越权调用指令</li>
        <li><span class="time">[01:15]</span> 大模型 API 鉴权校验通过，调用秘钥无异常</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.security-container { background: #ffffff; border: 1px solid #e2e8f0; border-radius: 12px; padding: 20px; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05); text-align: left; }
h3 { margin-top: 0; color: #1e293b; font-size: 16px; border-bottom: 1px solid #f1f5f9; padding-bottom: 8px; }
.desc { font-size: 12px; color: #64748b; }
.status-tag { color: #10b981; font-weight: bold; }
.metrics { display: flex; gap: 12px; margin-bottom: 15px; }
.card { flex: 1; background: #f8fafc; border: 1px solid #f1f5f9; padding: 10px; border-radius: 6px; }
.label { font-size: 11px; color: #64748b; display: block; }
.value { font-size: 18px; font-weight: bold; margin-top: 4px; display: block; }
.text-red { color: #ef4444; }
.text-green { color: #10b981; }
.log-box h4 { font-size: 12px; margin: 10px 0 6px 0; color: #475569; }
ul { list-style: none; padding: 0; margin: 0; font-size: 11px; color: #334155; line-height: 1.8; }
.time { color: #94a3b8; margin-right: 4px; }
</style>