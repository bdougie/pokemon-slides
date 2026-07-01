<template>
  <div class="slidev-layout is-dark section-layout h-full flex flex-col justify-center">
    <div class="eyebrow red mb-6" v-if="chapter">{{ chapter }}</div>
    <div class="sect-body"><slot /></div>
    <div class="rule" />
  </div>
</template>

<script setup>
defineProps({ chapter: { default: '' } })
</script>

<style scoped>
.section-layout { padding: 3.4rem 4rem; }
.sect-body :deep(h1) {
  font-family: var(--font-display);
  font-weight: 900;
  font-size: 3.4rem;
  line-height: .98;
  letter-spacing: -0.03em;
  color: var(--paper);
  max-width: 18ch;
  margin: 0;
}
.sect-body :deep(h1 .hl), .sect-body :deep(.hl) { color: var(--red); }
.sect-body :deep(p) { color: #c9b8a0; font-size: 1.1rem; margin-top: 1.3rem; max-width: 46ch; }
.sect-body :deep(strong) { color: var(--paper); }
.rule { height: 5px; width: 6rem; margin-top: 2rem; background: var(--red); }
</style>
