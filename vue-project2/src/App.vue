<script setup>
import BiliBanner from './components/BiliBanner.vue'

/**
 * 图层配置：moveX / moveY 分别控制水平/垂直最大偏移量(px)
 *
 * moveX：
 *  正值 = 跟随鼠标水平同向移动
 *  负值 = 与鼠标反向移动（产生景深视差感）
 *  0    = 水平静止
 *
 * moveY（实际偏移 = b × moveY，有符号，左右滑镜像对称）：
 *  正值 = 右滑时向下、左滑时向上（雪花/漂浮物用正值，形成弧线感）
 *  负值 = 右滑时向上、左滑时向下
 *  0    = 垂直静止
 *
 */
const layers = [
  // ── 远景：几乎静止 ─────────────────────────────────
  { src: '/images/layer_00.webp', moveX:  0,   moveY:  0  }, // 天空/纯色背景
  { src: '/images/layer_01.webp', moveX:  5,   moveY:  0  }, // 远景树后排
  { src: '/images/layer_02.webp', moveX: -8,   moveY:  0  }, // 远景树前排
  { src: '/images/layer_03.webp', moveX:  10,   moveY:  0  }, // 远景树
  { src: '/images/layer_04.webp', moveX:  0,   moveY:  0  }, // 静态装饰

  // ── 中远景：轻微晃动 ───────────────────────────────
  { src: '/images/layer_05.webp', moveX: -9,   moveY:  0  }, // 中远景
  { src: '/images/layer_06.webp', moveX:  6,   moveY:  0  }, // 候车站
  { src: '/images/layer_07.webp', moveX:  6,   moveY:  0  }, // 护栏
  { src: '/images/layer_08.webp', moveX: -8,   moveY:  0  }, // 天空
  { src: '/images/layer_09.webp', moveX:  0,   moveY:  0  }, // 静态
  { src: '/images/layer_10.webp', moveX:  0,   moveY:  0  }, // 静态

  // ── 中景 ───────────────────────────────────────────
  { src: '/images/layer_11.webp', moveX:  5,   moveY:  0  }, // 远景树
  { src: '/images/layer_12.webp', moveX:  8,   moveY:  0  }, // 柱子
  { src: '/images/layer_13.webp', moveX:  9,   moveY:  0  }, // 中景
  { src: '/images/layer_14.webp', moveX:  7,   moveY:  0  }, // 中景树左
  { src: '/images/layer_15.webp', moveX:  9,   moveY:  0  }, // 中景树右1
  { src: '/images/layer_16.webp', moveX:  9,   moveY:  0  }, // 中景树右2
  { src: '/images/layer_17.webp', moveX:  10,   moveY:  0  }, // 中景人物右侧树
  { src: '/images/layer_18.webp', moveX:  23,  moveY:  0  }, // 树

  // ── 中近景 ─────────────────────────────────────────
  { src: '/images/layer_19.webp', moveX:  9,   moveY:  0  }, // 中景人物
  { src: '/images/layer_20.webp', moveX:  0,   moveY:  0  }, // 静态
  { src: '/images/layer_21.webp', moveX:  16,   moveY:  0  }, // 前景左侧树
  { src: '/images/layer_22.webp', moveX:  0,   moveY:  0  }, // 静态
  { src: '/images/layer_23.webp', moveX:  8,   moveY:  10  }, // 雪花1

  // ── 前景：移动最大，与背景反向，景深感最强 ─────────
  { src: '/images/layer_24.webp', moveX:  9,   moveY:  0  }, // 鹿
  { src: '/images/layer_25.webp', moveX:  0,   moveY:  0  }, // 前景装饰
  { src: '/images/layer_26.webp', moveX: -26,  moveY:  0  }, // 前景树枝
  { src: '/images/layer_27.webp', moveX:  10,   moveY: 0  }, // 漂浮物
  { src: '/images/layer_28.webp', moveX:-10,   moveY: 0  }, // 漂浮物
  { src: '/images/layer_29.webp', moveX:-10,   moveY: 0  }, // 漂浮物
  { src: '/images/layer_30.webp', moveX: 26,   moveY: 0  }, // 前景树枝
  { src: '/images/layer_31.webp', moveX:  20,   moveY: 20 }, // 雪花2
]
</script>

<template>
  <BiliBanner bgSrc="/images/bg.png" :layers="layers" />
</template>

<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { background: #f1f2f3; }
</style>