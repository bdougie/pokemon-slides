<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'
import { useNav } from '@slidev/client'

// Shift+→ jumps to the next chapter, Shift+← to the previous one.
// Chapters are detected dynamically as the `layout: section` slides,
// so this keeps working if you add, remove, or reorder chapters.
const { slides, currentSlideNo, total, go } = useNav()

function chapterPages(): number[] {
  return slides.value
    .filter((r: any) => r.meta?.layout === 'section')
    .map((r: any) => r.no as number)
    .sort((a, b) => a - b)
}

function onKeydown(e: KeyboardEvent) {
  if (!e.shiftKey || e.repeat) return
  if (e.key !== 'ArrowRight' && e.key !== 'ArrowLeft') return

  const list = chapterPages()
  const cur = currentSlideNo.value
  const target =
    e.key === 'ArrowRight'
      ? list.find((p) => p > cur) ?? total.value
      : [...list].reverse().find((p) => p < cur) ?? 1

  // Preempt Slidev's default arrow navigation for this event.
  e.preventDefault()
  e.stopImmediatePropagation()
  go(target)
}

// Capture phase so we run before Slidev's own key handling.
onMounted(() => window.addEventListener('keydown', onKeydown, true))
onUnmounted(() => window.removeEventListener('keydown', onKeydown, true))
</script>

<template>
  <span style="display: none" />
</template>
