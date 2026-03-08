<template>

    <div class="game-wrapper">
      <!-- Loading overlay -->
      <div v-if="loading" class="loading">
        <div class="loading-bar" :class="{ indeterminate: !hasRealProgress }">
          <div class="fill" :style="hasRealProgress ? { width: progress + '%' } : {}"></div>
        </div>
        <p class="loading-text">
          {{ hasRealProgress ? `Loading ${progress}%` : 'Loading…' }}
        </p>
      </div>

      <canvas
          ref="canvasEl"
          id="godot-canvas"
          class="game-canvas"
          tabindex="0"
          :aria-busy="loading ? 'true' : 'false'"
      ></canvas>
    </div>

</template>

<script setup lang="ts">
import {onBeforeUnmount, onMounted, ref} from 'vue'

const canvasEl = ref<HTMLCanvasElement | null>(null)
const loading = ref(true)
const progress = ref(0)
const hasRealProgress = ref(false)

let cleanup: (() => void) | null = null
let godotReady: Promise<void> | null = null

function resizeCanvas(canvas: HTMLCanvasElement) {
  const cssW = Math.round(canvas.clientWidth)
  const cssH = Math.round(canvas.clientHeight)
  const dpr = Math.max(1, Math.floor(window.devicePixelRatio || 1))
  const w = cssW * dpr
  const h = cssH * dpr
  if (canvas.width !== w || canvas.height !== h) { canvas.width = w; canvas.height = h }
}

function loadGodotOnce(src: string) {
  if (godotReady) return godotReady
  godotReady = new Promise<void>((resolve, reject) => {
    const g: any = window as any
    if (typeof g.Engine === 'function' || typeof g.createEngine === 'function') {
      resolve(); return
    }
    const s = document.createElement('script')
    s.src = src
    s.defer = true
    s.onload = () => resolve()
    s.onerror = () => reject(new Error(`Failed to load ${src}`))
    document.head.appendChild(s)
  })
  return godotReady
}

onMounted(async () => {
  const canvas = canvasEl.value!
  resizeCanvas(canvas)

  const base = import.meta.env.BASE_URL // '/' locally, '/<repo>/' on GH Pages
  await loadGodotOnce(`${base}game/Kekkon.js`)

  const g: any = window as any

  // If the loader supports progress, wire it up
  const opts: Record<string, any> = {
    canvas,
    canvasResizePolicy: 0,
    executable: `${base}game/Kekkon`,
    onProgress: (current: number, total: number) => {
      if (!total || total <= 0) return
      hasRealProgress.value = true
      progress.value = Math.max(0, Math.min(100, Math.round((current / total) * 100)))
    },
  }

  // Start engine (supports both ctor and factory styles)
  const engine =
      typeof g.Engine === 'function' ? new g.Engine(opts) :
          typeof g.createEngine === 'function' ? await g.createEngine(opts) :
              (() => { throw new Error('Godot loader not found') })()

  await engine.startGame()
  document.title = 'Konishi & Ofstad'
  // Done loading
  progress.value = 100
  loading.value = false

  // Autofocus and keep focus (no outline)
  canvas.focus()
  canvas.addEventListener('blur', () => { if (document.hasFocus()) canvas.focus() })

  // Keep crisp on resize / DPR change
  const onResize = () => resizeCanvas(canvas)
  const ro = new ResizeObserver(onResize)
  ro.observe(canvas)
  window.addEventListener('resize', onResize)
  cleanup = () => { ro.disconnect(); window.removeEventListener('resize', onResize) }
})

onBeforeUnmount(() => cleanup?.())

</script>

<style scoped>
.game-wrapper {
  position: relative;
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0;
}

/* Default (desktop/tablet): wide 16:9 */
.game-canvas {
  width: 80vw;
  max-width: 1920px;
  max-height: 700px;
  aspect-ratio: 16 / 9;
  background: #000;
  border: none;
  outline: none;
  border-radius: 10px;
  box-shadow: 0 8px 24px rgba(0,0,0,.25);
  display: block;
}

/* Mobile portrait: taller with a little margin */
@media (max-width: 900px) and (orientation: portrait) {
  .game-wrapper {
    padding: 0 0.5rem;           /* small padding around game */
  }

  .game-canvas {
    width: 94vw;                    /* small margin on sides */
    height: auto;
    aspect-ratio: 9 / 16;           /* portrait ratio */
    max-height: 90vh;               /* leave some space top/bottom */
    border-radius: 8px;
  }
}

/* Prevent touch scrolling inside the canvas */
@media (hover: none) and (pointer: coarse) {
  .game-canvas { touch-action: none; }
}

/* Loading overlay */
.loading {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  gap: .75rem;
  justify-content: center;
  align-items: center;
  z-index: 2;
  pointer-events: none;
}

.loading-bar {
  position: relative;
  width: min(60vw, 420px);
  height: 10px;
  background: rgba(255,255,255,0.15);
  border-radius: 999px;
  overflow: hidden;
  box-shadow: inset 0 0 6px rgba(0,0,0,0.35);
}

.loading-bar .fill {
  height: 100%;
  width: 0%;
  background: linear-gradient(90deg, #002a78, #000143);
  border-radius: 999px;
  transition: width .2s ease;
}

/* Indeterminate animation if no real progress available */
.loading-bar.indeterminate .fill {
  position: absolute;
  width: 30%;
  left: -30%;
  animation: slide 1.2s infinite ease-in-out;
}

@keyframes slide {
  0%   { left: -30%; }
  50%  { left: 50%;  }
  100% { left: 110%; }
}

.loading-text {
  color: #000;
  font-size: .95rem;
}


</style>
