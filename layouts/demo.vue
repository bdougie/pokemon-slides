<template>
  <div class="slidev-layout demo-layout h-full flex flex-col justify-center">
    <div class="flex items-center gap-3 mb-7">
      <span class="dot" />
      <span class="eyebrow red blink">● LIVE DEMO</span>
    </div>
    <div class="demo-body"><slot /></div>
  </div>
</template>

<style scoped>
.demo-layout { padding: 3.4rem 4rem; }
.dot { width: .65rem; height: .65rem; border-radius: 50%; background: var(--red); box-shadow: 0 0 0 4px rgba(191,66,49,.18); }
.blink { animation: blink 1.5s steps(2,start) infinite; }
@keyframes blink { 50% { opacity: .4; } }
.demo-body :deep(h1) { font-family: var(--font-display); font-weight: 800; font-size: 2.7rem; letter-spacing: -0.02em; }
.demo-body :deep(ul) { margin-top: .9rem; }
</style>
