<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

interface Particle {
  x: number
  y: number
  size: number
  color: string
  speedX: number
  speedY: number
  alpha: number
  decay: number
  gravity: number
  trail: { x: number; y: number }[]
  hue: number
  brightness: number
  saturation: number
}

const canvasRef = ref<HTMLCanvasElement | null>(null)
let particles: Particle[] = []
let animationFrameId: number

const getRandomColor = () => {
  const colorSchemes = [
    // 经典烟花色系
    { hue: 0, saturation: 100, brightness: 60 },      // 鲜红色
    { hue: 60, saturation: 100, brightness: 60 },     // 金黄色
    { hue: 120, saturation: 85, brightness: 55 },     // 翠绿色
    { hue: 240, saturation: 95, brightness: 65 },     // 皇家蓝
    { hue: 300, saturation: 95, brightness: 65 },     // 紫罗兰
    
    // 柔和色系
    { hue: 20, saturation: 85, brightness: 70 },      // 珊瑚橙
    { hue: 190, saturation: 90, brightness: 65 },     // 青蓝色
    { hue: 280, saturation: 70, brightness: 75 },     // 淡紫色
    { hue: 340, saturation: 80, brightness: 75 },     // 粉红色
    
    // 金属色系
    { hue: 45, saturation: 100, brightness: 65 },     // 金色
    { hue: 0, saturation: 0, brightness: 80 },        // 银色
    { hue: 30, saturation: 90, brightness: 55 },      // 青铜色
    
    // 梦幻色系
    { hue: 160, saturation: 85, brightness: 60 },     // 绿松石
    { hue: 210, saturation: 85, brightness: 70 },     // 天蓝色
    { hue: 320, saturation: 75, brightness: 75 },     // 玫瑰粉
    
    // 霓虹色系
    { hue: 180, saturation: 100, brightness: 65 },    // 霓虹青
    { hue: 300, saturation: 100, brightness: 70 },    // 霓虹紫
    { hue: 30, saturation: 100, brightness: 65 }      // 霓虹橙
  ]
  
  // 随机选择两个颜色进行混合
  const scheme1 = colorSchemes[Math.floor(Math.random() * colorSchemes.length)]
  let scheme2 = colorSchemes[Math.floor(Math.random() * colorSchemes.length)]
  
  // 确保两个颜色不完全相同
  while (scheme2.hue === scheme1.hue) {
    scheme2 = colorSchemes[Math.floor(Math.random() * colorSchemes.length)]
  }
  
  // 随机混合两种颜色
  const mixRatio = Math.random()
  const hue = scheme1.hue * mixRatio + scheme2.hue * (1 - mixRatio)
  
  return {
    hue: hue + (Math.random() * 15 - 7.5),  // 添加微小的随机色调变化
    saturation: Math.min(100, (scheme1.saturation + scheme2.saturation) / 2 + (Math.random() * 10 - 5)),
    brightness: (scheme1.brightness + scheme2.brightness) / 2 + (Math.random() * 10 - 5)
  }
}

const createParticles = (x: number, y: number) => {
  const particleCount = 120 
  const baseColor = getRandomColor()

  // 爆炸
  for (let i = 0; i < particleCount; i++) {
    const angle = (Math.PI * 2 * i) / particleCount
    const velocity = 3 + Math.random() * 4  // Random velocity
    const particleHue = baseColor.hue + (Math.random() * 20 - 10)
    
    particles.push({
      x,
      y,
      size: 1.5 + Math.random() * 2,
      color: `hsl(${particleHue}, ${baseColor.saturation}%, ${baseColor.brightness}%)`,
      speedX: Math.cos(angle) * velocity * (0.8 + Math.random() * 0.4),
      speedY: Math.sin(angle) * velocity * (0.8 + Math.random() * 0.4),
      alpha: 1,
      decay: 0.005 + Math.random() * 0.01,
      gravity: 0.15 + Math.random() * 0.05,
      trail: [],
      hue: particleHue,
      brightness: baseColor.brightness,
      saturation: baseColor.saturation
    })
  }

  // 散射
  for (let i = 0; i < 30; i++) {
    const angle = Math.random() * Math.PI * 2
    const velocity = 1 + Math.random() * 2
    particles.push({
      x,
      y,
      size: 0.5 + Math.random(),
      color: `hsl(${baseColor.hue}, ${baseColor.saturation}%, ${baseColor.brightness}%)`,
      speedX: Math.cos(angle) * velocity,
      speedY: Math.sin(angle) * velocity,
      alpha: 1,
      decay: 0.01 + Math.random() * 0.015,
      gravity: 0.1,
      trail: [],
      hue: baseColor.hue,
      brightness: baseColor.brightness,
      saturation: baseColor.saturation
    })
  }
}

  const animate = () => {
  if (!canvasRef.value) return

  const ctx = canvasRef.value.getContext('2d')
  if (!ctx) return

  // 残影
  ctx.fillStyle = 'rgba(0, 0, 0, 0.2)'
  ctx.fillRect(0, 0, canvasRef.value.width, canvasRef.value.height)

  particles.forEach((particle, index) => {
    // update speed and position
    particle.x += particle.speedX
    particle.y += particle.speedY
    particle.speedY += particle.gravity
    particle.alpha -= particle.decay

    // update trail
    particle.trail.push({ x: particle.x, y: particle.y })
    if (particle.trail.length > 5) {
      particle.trail.shift()
    }

    // canvas trail
    if (particle.trail.length > 1) {
      ctx.beginPath()
      ctx.moveTo(particle.trail[0].x, particle.trail[0].y)
      for (let i = 1; i < particle.trail.length; i++) {
        ctx.lineTo(particle.trail[i].x, particle.trail[i].y)
      }
      ctx.strokeStyle = `hsla(${particle.hue}, ${particle.saturation}%, ${particle.brightness}%, ${particle.alpha * 0.5})`
      ctx.lineWidth = particle.size * 0.5
      ctx.stroke()
    }

    // canvas particle
    ctx.globalAlpha = particle.alpha
    ctx.fillStyle = `hsla(${particle.hue}, ${particle.saturation}%, ${particle.brightness}%, ${particle.alpha})`
    ctx.beginPath()
    ctx.arc(particle.x, particle.y, particle.size, 0, Math.PI * 2)
    ctx.fill()

    // 光晕
    const gradient = ctx.createRadialGradient(
      particle.x, particle.y, 0,
      particle.x, particle.y, particle.size * 2
    )
    gradient.addColorStop(0, `hsla(${particle.hue}, ${particle.saturation}%, ${particle.brightness}%, ${particle.alpha * 0.5})`)
    gradient.addColorStop(1, `hsla(${particle.hue}, ${particle.saturation}%, ${particle.brightness}%, 0)`)
    ctx.fillStyle = gradient
    ctx.beginPath()
    ctx.arc(particle.x, particle.y, particle.size * 2, 0, Math.PI * 2)
    ctx.fill()

    // remove dead particles
    if (particle.alpha <= 0) {
      particles.splice(index, 1)
    }
  });

  animationFrameId = requestAnimationFrame(animate);
}

const handleClick = (e: MouseEvent) => {
  if (!canvasRef.value) return
  const rect = canvasRef.value.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  createParticles(x, y)
}

const updateCanvasSize = () => {
  if (!canvasRef.value) return
  canvasRef.value.width = window.innerWidth
  canvasRef.value.height = window.innerHeight
}

onMounted(() => {
  updateCanvasSize()
  window.addEventListener('resize', updateCanvasSize)
  animate()
})

onUnmounted(() => {
  window.removeEventListener('resize', updateCanvasSize)
  cancelAnimationFrame(animationFrameId)
})
</script>

<template>
  <canvas ref="canvasRef" @click="handleClick" class="fireworks-canvas"></canvas>
</template>

<style scoped>
.fireworks-canvas {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: auto;
  z-index: 999;
}
</style>
