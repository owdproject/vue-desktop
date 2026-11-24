<script setup lang="ts">
import { DesktopPointerSession, type Desktop, type WindowInstance } from '@owdproject/core'
import { computed, onMounted, onUnmounted, shallowRef, useTemplateRef } from 'vue'

const { instance, desktop } = defineProps<{ instance: WindowInstance, desktop: Desktop }>()
defineEmits<{ close: [] }>()

const snapshot = shallowRef(instance.getSnapshot())
const unsubscribe = instance.subscribe(state => { snapshot.value = state })
const closeButton = useTemplateRef<HTMLButtonElement>('closeButton')

onMounted(() => closeButton.value?.focus())
onUnmounted(unsubscribe)

const bounds = computed(() => ({
  x: snapshot.value.position?.x ?? 0,
  y: snapshot.value.position?.y ?? 0,
  width: Number(snapshot.value.size?.width ?? 420),
  height: Number(snapshot.value.size?.height ?? 235),
}))

function drag(event: PointerEvent) {
  desktop.windows.focusWindow(instance.id)
  DesktopPointerSession.startDrag(event, {
    initialBounds: bounds.value,
    workarea: desktop.workspaces.workarea,
    onStart: () => instance.startDrag(),
    onMove: ({ x, y }) => instance.setPosition({ x, y }),
    onEnd: (position, snapZone) => {
      instance.stopDrag()
      if (snapZone) instance.setLayout(snapZone, desktop.workspaces.workarea)
      else instance.setPosition({ x: position.x, y: position.y })
    },
    onCancel: () => instance.stopDrag(),
  })
}

function resize(event: PointerEvent) {
  desktop.windows.focusWindow(instance.id)
  const update = ({ width, height }: { width: number, height: number }) => {
    instance.setSize({ width: Math.max(240, width), height: Math.max(175, height) })
  }
  DesktopPointerSession.startResize(event, {
    direction: 'se',
    initialBounds: bounds.value,
    workarea: desktop.workspaces.workarea,
    onStart: () => instance.startResize(),
    onMove: update,
    onEnd: (size) => {
      instance.stopResize()
      update(size)
    },
    onCancel: () => instance.stopResize(),
  })
}
</script>

<template>
  <section
    class="desktop-window"
    aria-label="About Vue Desktop"
    :style="{
      left: bounds.x + 'px',
      top: bounds.y + 'px',
      width: bounds.width + 'px',
      height: bounds.height + 'px',
    }"
  >
    <div class="titlebar">
      <span data-owd-drag-handle @pointerdown="drag">About</span>
      <button ref="closeButton" type="button" aria-label="Close About" @click="$emit('close')">×</button>
    </div>
    <div class="window-content">
      <h2>A desktop in Vue</h2>
      <p>The Vue components follow snapshots from the OWD core. The runtime handles window geometry and pointer interactions.</p>
    </div>
    <span class="resize-handle" @pointerdown="resize"></span>
  </section>
</template>
