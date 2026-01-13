<template>
  <div class="game-wrapper">
    <h1>🐍 Snake Game</h1>

    <div class="top-bar">
      <span>Score: {{ score }}</span>
      <button @click="toggleTheme">
        {{ theme === 'neon' ? '🌙 Dark' : '🌈 Neon' }}
      </button>
    </div>

    <canvas
      ref="canvasRef"
      :width="canvasSize"
      :height="canvasSize"
    ></canvas>

    <div v-if="gameOver" class="game-over">
      <h2>💀 Game Over</h2>
      <button @click="resetGame">Reiniciar</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

/* ===== CONFIG ===== */
const canvasSize = 400
const tiles = 20
const tileSize = canvasSize / tiles

/* ===== STATE ===== */
const canvasRef = ref(null)
let ctx

let snake = []
let direction = 'right'
let nextDirection = 'right'
let food = {}
let score = ref(0)
let gameOver = ref(false)
let theme = ref('neon')

let gameInterval
let animationId
let time = 0

/* ===== UTILS ===== */
function randomPosition() {
  return Math.floor(Math.random() * tiles)
}

/* ===== GAME LOGIC ===== */
function resetGame() {
  snake = [{ x: 10, y: 10 }]
  direction = 'right'
  nextDirection = 'right'
  food = {
    x: randomPosition(),
    y: randomPosition()
  }
  score.value = 0
  gameOver.value = false
  time = 0

  clearInterval(gameInterval)
  cancelAnimationFrame(animationId)

  gameInterval = setInterval(updateGame, 120)
  animate()
}

function updateGame() {
  if (gameOver.value) return

  direction = nextDirection

  const head = { ...snake[0] }

  if (direction === 'right') head.x++
  if (direction === 'left') head.x--
  if (direction === 'up') head.y--
  if (direction === 'down') head.y++

  // wall collision
  if (
    head.x < 0 ||
    head.y < 0 ||
    head.x >= tiles ||
    head.y >= tiles
  ) {
    endGame()
    return
  }

  // self collision
  if (snake.some(p => p.x === head.x && p.y === head.y)) {
    endGame()
    return
  }

  snake.unshift(head)

  // eat food
  if (head.x === food.x && head.y === food.y) {
    score.value++
    food = {
      x: randomPosition(),
      y: randomPosition()
    }
  } else {
    snake.pop()
  }
}

function endGame() {
  gameOver.value = true
  clearInterval(gameInterval)
}

/* ===== DRAW ===== */
function drawBackground() {
  ctx.fillStyle = theme.value === 'neon' ? '#050505' : '#111'
  ctx.fillRect(0, 0, canvasSize, canvasSize)

  ctx.strokeStyle = 'rgba(255,255,255,0.05)'
  for (let i = 0; i <= tiles; i++) {
    ctx.beginPath()
    ctx.moveTo(i * tileSize, 0)
    ctx.lineTo(i * tileSize, canvasSize)
    ctx.stroke()

    ctx.beginPath()
    ctx.moveTo(0, i * tileSize)
    ctx.lineTo(canvasSize, i * tileSize)
    ctx.stroke()
  }
}

function drawSnake() {
  ctx.shadowBlur = theme.value === 'neon' ? 15 : 0
  ctx.shadowColor = '#00fff7'

  snake.forEach((p, i) => {
    ctx.fillStyle =
      i === 0 ? '#00fff7' : '#00cc7a'

    ctx.fillRect(
      p.x * tileSize,
      p.y * tileSize,
      tileSize,
      tileSize
    )
  })

  ctx.shadowBlur = 0
}

function drawFood() {
  const pulse = Math.sin(time / 10) * 2 + 2

  ctx.shadowBlur = 12
  ctx.shadowColor = '#ff4757'
  ctx.fillStyle = '#ff4757'

  ctx.fillRect(
    food.x * tileSize + pulse / 2,
    food.y * tileSize + pulse / 2,
    tileSize - pulse,
    tileSize - pulse
  )

  ctx.shadowBlur = 0
}

function animate() {
  time++
  drawBackground()
  drawFood()
  drawSnake()
  animationId = requestAnimationFrame(animate)
}

/* ===== INPUT ===== */
function handleKey(e) {
  if (e.key === 'ArrowUp' && direction !== 'down') nextDirection = 'up'
  if (e.key === 'ArrowDown' && direction !== 'up') nextDirection = 'down'
  if (e.key === 'ArrowLeft' && direction !== 'right') nextDirection = 'left'
  if (e.key === 'ArrowRight' && direction !== 'left') nextDirection = 'right'
}

function toggleTheme() {
  theme.value = theme.value === 'neon' ? 'dark' : 'neon'
}

/* ===== LIFECYCLE ===== */
onMounted(() => {
  ctx = canvasRef.value.getContext('2d')
  window.addEventListener('keydown', handleKey)
  resetGame()
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKey)
  clearInterval(gameInterval)
  cancelAnimationFrame(animationId)
})
</script>

<style scoped>
.game-wrapper {
  min-height: 100vh;
  background: #000;
  color: #fff;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  font-family: system-ui, sans-serif;
}

canvas {
  border-radius: 12px;
}

.top-bar {
  width: 400px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

button {
  padding: 6px 12px;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  background: #00fff7;
  color: #000;
  font-weight: bold;
}

.game-over {
  position: absolute;
  background: rgba(0,0,0,0.85);
  padding: 20px 30px;
  border-radius: 16px;
  text-align: center;
}
</style>
