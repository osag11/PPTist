<template>
  <SvgWrapper class="grid-lines">
    <path 
      :style="{
        transform: `scale(${canvasScale})`,
      }" 
      :d="path" 
      fill="none" 
      :stroke="gridColor" 
      stroke-width="0.3" 
      stroke-dasharray="5"
    ></path>
  </SvgWrapper>
</template>

<script lang="ts">
import { defineComponent, computed } from 'vue'
import tinycolor from 'tinycolor2'
import { useStore } from '@/store'
import { VIEWPORT_SIZE } from '@/configs/canvas'
import { SlideBackground } from '@/types/slides'

export default defineComponent({
  name: 'grid-lines',
  setup() {
    const store = useStore()
    const canvasScale = computed(() => store.state.canvasScale)
    const viewportRatio = computed(() => store.state.viewportRatio)
    const background = computed<SlideBackground | undefined>(() => store.getters.currentSlide?.background)

    // 计算网格线的颜色，避免与背景的颜色太接近
    const gridColor = computed(() => {
      const bgColor = background.value?.color || '#fff'
      const colorList = ['#000', '#fff']
      return tinycolor.mostReadable(bgColor, colorList, { includeFallbackColors: true }).setAlpha(.5).toRgbString()
    })

    const gridSize = 50

    // 计算网格路径
    const getPath = () => {
      const maxX = VIEWPORT_SIZE
      const maxY = VIEWPORT_SIZE * viewportRatio.value

      let path = ''
      for (let i = 0; i <= Math.floor(maxY / gridSize); i++) {
        path += `M0 ${i * gridSize} L${maxX} ${i * gridSize} `
      }
      for (let i = 0; i <= Math.floor(maxX / gridSize); i++) {
        path += `M${i * gridSize} 0 L${i * gridSize} ${maxY} `
      }
      return path
    }

    return {
      canvasScale,
      gridColor,
      width: VIEWPORT_SIZE,
      height: VIEWPORT_SIZE * viewportRatio.value,
      path: getPath(),
    }
  },
})
</script>

<style lang="scss" scoped>
.grid-lines {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  overflow: visible;
  z-index: 999;
  pointer-events: none;
}
</style>