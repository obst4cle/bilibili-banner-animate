<script setup>
import { ref, reactive, onUnmounted } from 'vue'

const props = defineProps({
  /** 底图路径（不透明宽幅背景） */
  bgSrc: { type: String, default: '/images/bg.png' },
  /**
   * 图层数组，每项：
   * {
   *   src:    string   图片路径
   *   type:   string   'img'(默认) | 'video'
   *   moveX:  number   水平最大偏移px
   *                    实际偏移 = b × moveX（b 为 -1~1 的位移比例）
   *                    正值=随鼠标同向，负值=反向（产生景深感）
   *   moveY:  number   垂直最大偏移px
   *                    实际偏移 = b × moveY（有符号，左右镜像对称）
   *                    正值=右滑时向下、左滑时向上（雪花用正值）
   *                    负值=右滑时向上、左滑时向下
   *   blur:   number   最大高斯模糊px（可选，默认0不模糊）
   *                    实际模糊 = |b| × blur
   * }
   */
  layers: { type: Array, default: () => [] },
})

const bannerRef = ref(null)

/**
 * b：鼠标相对「进入点」的位移比例，范围 -1 ~ 1
 * b = (当前鼠标X - 进入时鼠标X) / 容器宽度
 */
let b = 0
let anchorX = 0  // 事件①：mouseenter 时记录的起始 X 坐标

// cur：对 b 做帧间缓动后的实际驱动值，避免跳变
const cur = reactive({ x: 0, y: 0 })

let rafId = null      // 缓动动画帧 id
let easeRafId = null  // 事件③回弹动画帧 id

function lerp(a, b, t) { return a + (b - a) * t }

// 将 cur 平滑追近目标 b，动画结束后自动停止
function tick() {
  cur.x = lerp(cur.x, b, 0.08)
  cur.y = lerp(cur.y, Math.abs(b), 0.08)
  if (Math.abs(cur.x - b) < 0.0005 && Math.abs(cur.y - Math.abs(b)) < 0.0005) {
    cur.x = b
    cur.y = Math.abs(b)
    rafId = null
    return
  }
  rafId = requestAnimationFrame(tick)
}

function startTick() {
  if (!rafId) rafId = requestAnimationFrame(tick)
}

// ── 事件①：鼠标进入 banner ────────────────────────────────────────────────
function onMouseEnter(e) {
  // 取消正在进行的回弹动画
  if (easeRafId) { cancelAnimationFrame(easeRafId); easeRafId = null }
  // 记录此刻的 X 作为后续计算位移的基准点
  anchorX = e.clientX
}

// ── 事件②：鼠标在 banner 内移动 ──────────────────────────────────────────
function onMouseMove(e) {
  const rect = bannerRef.value.getBoundingClientRect()
  // 相对进入点的位移比例，钳制到 -1~1
  b = Math.min(1, Math.max(-1, (e.clientX - anchorX) / rect.width))
  startTick()
}

// ── 事件③：鼠标离开，200ms 线性回弹到 0 ─────────────
function onMouseLeave() {
  if (rafId) { cancelAnimationFrame(rafId); rafId = null }

  const startTime = performance.now()
  const startB = b
  const DURATION = 200  // ms

  const ease = (now) => {
    const elapsed = now - startTime
    if (elapsed < DURATION) {
      // 线性衰减：b = startB × (1 - elapsed/200)，
      b = startB * (1 - elapsed / DURATION)
      startTick()
      easeRafId = requestAnimationFrame(ease)
    } else {
      b = 0
      startTick()
      easeRafId = null
    }
  }
  easeRafId = requestAnimationFrame(ease)
}

onUnmounted(() => {
  if (rafId) cancelAnimationFrame(rafId)
  if (easeRafId) cancelAnimationFrame(easeRafId)
})

// ── 图层样式：由缓动后的 cur 驱动平移和模糊 ─────────────────────────────
function layerStyle(layer) {
  const tx = cur.x * (layer.moveX ?? 0)          // 水平偏移，有符号跟随鼠标方向
  const ty = cur.x * (layer.moveY ?? 0)          // 垂直偏移，有符号：左右滑镜像对称
  const blurPx = cur.y * (layer.blur ?? 0)       // 高斯模糊，用 |b|：偏离越多越模糊

  const style = {}
  if (tx !== 0 || ty !== 0)
    style.transform = `translate(${tx}px, ${ty}px)`
  if (blurPx > 0.01)
    style.filter = `blur(${blurPx.toFixed(2)}px)`
  if (tx !== 0 || ty !== 0 || blurPx > 0.01)
    style.willChange = 'transform, filter'
  return style
}
</script>

<template>
  <div
    class="bili-header__banner"
    ref="bannerRef"
    @mouseenter="onMouseEnter"
    @mousemove="onMouseMove"
    @mouseleave="onMouseLeave"
  >
    <img class="banner-bg" :src="props.bgSrc" alt="" draggable="false" />

    <!-- 动态多图层 -->
    <div class="animated-banner">
      <div
        v-for="(layer, i) in props.layers"
        :key="i"
        class="layer"
        :style="layerStyle(layer)"
      >
        <video
          v-if="layer.type === 'video'"
          :src="layer.src"
          loop autoplay muted playsinline
        />
        <img
          v-else
          :src="layer.src"
          alt=""
          draggable="false"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.bili-header__banner {
  position: relative;
  width: 100%;
  overflow: hidden;
  cursor: default;
  user-select: none;
}

/* 底图：宽度铺满，高度自适应，决定容器尺寸 */
.banner-bg {
  display: block;
  width: 100%;
  height: auto;
  pointer-events: none;
  user-select: none;
}



/* 动态层容器 */
.animated-banner {
  position: absolute;
  inset: 0;
  z-index: 1;
  pointer-events: none;
}

/* 每个动态图层：绝对定位铺满，内容居中 */
.layer {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

/*
  图层图片：宽度铺满容器宽度，高度自适应
  比容器高意味着上下有余量，移动时不露白
*/
.layer img,
.layer video {
  width: 100%;
  height: auto;
  max-height: none;
  display: block;
  pointer-events: none;
}



</style>
