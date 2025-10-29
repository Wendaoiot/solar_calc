<template>
  <div class="app">
    <header class="header">
      <h1>户外设备 太阳能 & 电池 计算器</h1>
      <p class="subtitle">前端小工具 — 估算所需太阳能板与电池容量</p>
    </header>

    <main class="main">
      <Calculator />
    </main>

    <footer class="footer">
      <div>小工具 · 无后端 · Vue + Vite</div>
      <div style="margin-top:6px;font-size:13px;color:var(--muted)">
        作者：<a href="http://www.wendaoiot.com" target="_blank" rel="noopener" style="color:var(--accent)">wendaoiot</a>
        &nbsp;·&nbsp; & Chat GPT-5 mini
        <a href="http://www.wendaoiot.com/?p=636" target="_blank" rel="noopener" style="color:var(--accent)">一站式户外设备供电解决方案</a>
      </div>
    </footer>
  </div>
</template>

<script setup>
import Calculator from './components/Calculator.vue'
</script>

<style>
:root{--accent:#0b86ff;--card:#fff;--muted:#666}
*{box-sizing:border-box}
body,html,#app{height:100%;margin:0;font-family:Segoe UI, Roboto, "PingFang SC", "Microsoft Yahei", Arial}
.app{min-height:100vh;display:flex;flex-direction:column;background:#f5f7fb;padding:16px}
.header{max-width:900px;margin:0 auto 16px;text-align:center}
.subtitle{color:var(--muted);margin-top:4px}
.main{flex:1;max-width:900px;margin:0 auto;width:100%}
.footer{text-align:center;color:var(--muted);font-size:13px;margin-top:20px}

</style>
