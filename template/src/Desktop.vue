<script setup lang="ts">
import { createDesktop } from '@owdproject/core'
import { nextTick, onMounted, onUnmounted, shallowRef, useTemplateRef } from 'vue'
import DesktopWindow from './DesktopWindow.vue'

const top = 64
const workarea = () => ({ x: 0, y: top, width: innerWidth, height: Math.max(0, innerHeight - top) })
const desktop = createDesktop({ workarea: workarea() })
const snapshot = shallowRef(desktop.getSnapshot())
const launcher = useTemplateRef<HTMLButtonElement>('launcher')
const unsubscribe = desktop.subscribe(value => { snapshot.value = value })
const updateWorkarea = () => desktop.workspaces.setWorkarea(workarea())

onMounted(() => window.addEventListener('resize', updateWorkarea))
onUnmounted(() => {
  unsubscribe()
  window.removeEventListener('resize', updateWorkarea)
})

function openAbout() {
  const window = desktop.windows.get('about')
  if (window) {
    desktop.windows.focusWindow(window.id)
    return
  }

  const width = Math.min(420, innerWidth - 32)
  const x = Math.max(16, innerWidth - width - Math.max(16, innerWidth * 0.07))
  const y = Math.max(top + 24, Math.min(innerHeight * 0.49, innerHeight - 251))
  desktop.windows.createWindow({
    id: 'about',
    model: 'about',
    config: {
      title: 'About',
      size: { width, height: 235, minWidth: 240, minHeight: 175 },
      position: { x: Math.round(x), y: Math.round(y) },
      resizable: true,
    },
  })
}

async function closeAbout() {
  desktop.windows.closeWindow('about')
  await nextTick()
  launcher.value?.focus()
}
</script>

<template>
  <header>
    <a href="https://owdproject.org">Open Web Desktop</a>
    <button ref="launcher" type="button" @click="openAbout">Open About</button>
  </header>
  <main aria-labelledby="workspace-title">
    <div class="workspace-intro">
      <p class="framework-label">Vue.js experiment</p>
      <h1 id="workspace-title">Vue Desktop</h1>
      <p>Start with a window. Build the rest in Vue.</p>
    </div>
    <DesktopWindow
      v-for="instance in snapshot.windows"
      :key="instance.id"
      :instance
      :desktop
      @close="closeAbout"
    />
  </main>
</template>
