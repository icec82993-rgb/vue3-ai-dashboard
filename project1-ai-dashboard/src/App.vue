<template>  //Vue 的核心语法标准。几乎所有 Vue 项目的单文件组件（.vue 文件）都是这么开头的。
//浏览器原生其实是不认识 .vue 文件的。Vue 通过打包工具（如 Vite 或 Webpack中的编译器 把标签包裹的内容，编译成极其高效的 JavaScript 虚拟 DOM  VNode 渲染函数。
//它告诉 Vue “这里面装的是纯粹的 HTML 结构，你帮我把它画到浏览器屏幕上
  <div class="dashboard-layout">  //网页开发的基本逻辑就是：先用 div 搬出一个个无辜、透明的空箱子来划分网页的物理结构
    //马上用 class 给这些箱子挂上不同的身份
    //最后在 CSS 里根据这些 class 狗牌，精准地给不同的箱子上色、定位、变形
    <div class="left-wing">   //两栏式（双列）的网页布局
      <ChatWindow />  //大写字母开头，/结尾。独立文件
    </div>
    <div class="right-wing">  //一旦开启 Flex 大箱子内部的前两个直接子  div 就会像听到了口令一样，瞬间变成横向并排、左一个右一个地排开。
      <SecurityPanel />  //同样是大写字母开头，/结尾。独立文件，安全面板，独立封装的安全中控/指标监控自定义组件
    </div>
  </div>
</template>

<script setup>  //Vue 3 的工业级逻辑特区
import ChatWindow from './components/ChatWindow.vue'
import SecurityPanel from './components/SecurityPanel.vue'  //引用
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
//24行代码，（左翼箱子）占了 2 份宽度
//25行代码，（右翼箱子）占了 1 份宽度
//21.CSS 样式代码
//22.所有浏览器（Chrome、Safari 等）默认都会给网页自带大约 8px 的外边距。如果不写 margin: 0 把它清零，你的网页四周就会露出一圈丑陋的白边。这一行叫样式重置（Reset）。
//22.font-family: system-ui, sans-serif; 这一行是字体设置，system-ui 是一个特殊的字体族，它会自动使用用户操作系统的默认界面字体。这种设置可以让你的网页在不同平台上看起来更自然和一致。sans-serif 是一个通用字体族，表示无衬线字体，如果系统默认字体不可用，就会回退到无衬线字体。
23.//Flexbox 布局，最大宽度 1100px，水平居中，左右内边距 20px，子元素间距 20px
27//响应式设计：当屏幕宽度小于等于 768px 时，调整布局为单列,最大宽度为 768 像素
28 flex-direction: column;横向排版方向扭转成了 column（纵向排版