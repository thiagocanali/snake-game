<template>
  <div class="game-wrapper" :class="theme">
    <h1>🐍 SNAKE FURY</h1>

    <div class="top-bar">
      <span>Score: {{ score }}</span>
      <span v-if="fury">🔥 FURY MODE</span>
      <button @click="toggleTheme">
        {{ theme === 'neon' ? '🌙 Dark' : '🌈 Neon' }}
      </button>
    </div>

    <div class="canvas-wrap" :class="{ shake }">
      <canvas
        ref="canvas"
        :width="canvasSize"
        :height="canvasSize"
      ></canvas>
    </div>

    <div v-if="gameOver" class="overlay">
      <div class="game-over">
        <h2>💀 GAME OVER</h2>
        <p>Score: {{ score }}</p>
        <button @click="resetGame">Reiniciar</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'

/* ===== CONFIG ===== */
const canvasSize = 420
const tileSize = 20
const tiles = canvasSize / tileSize

const baseSpeed = 130
const furySpeed = 75

/* ===== REFS ===== */
const canvas = ref(null)
let ctx
let loop

/* ===== STATE ===== */
const score = ref(0)
const gameOver = ref(false)
const theme = ref('neon')
const fury = ref(false)
const shake = ref(false)

let snake = []
let food = {}
let specialFood = null
let direction = { x: 1, y: 0 }
let nextDirection = { x: 1, y: 0 }
let particles = []

let speed = baseSpeed

/* ===== HELPERS ===== */
function rand(max) {
  return Math.floor(Math.random() * max)
}

function spawnFood() {
  food = { x: rand(tiles), y: rand(tiles) }

  if (Math.random() < 0.25) {
    specialFood = { x: rand(tiles), y: rand(tiles) }
  }
}

function spawnParticles(x, y, color) {
  for (let i = 0; i < 12; i++) {
    particles.push({
      x,
      y,
      vx: (Math.random() - 0.5) * 4,
      vy: (Math.random() - 0.5) * 4,
      life: 20,
      color
    })
  }
}

function triggerShake() {
  shake.value = true
  setTimeout(() => (shake.value = false), 120)
}

/* ===== GAME ===== */
function resetGame() {
  snake = [
    { x: 10, y: 10 },
    { x: 9, y: 10 },
    { x: 8, y: 10 }
  ]

  direction = { x: 1, y: 0 }
  nextDirection = { x: 1, y: 0 }

  score.value = 0
  fury.value = false
  gameOver.value = false
  particles = []
  speed = baseSpeed

  spawnFood()

  clearInterval(loop)
  loop = setInterval(gameLoop, speed)
}

function collision(head) {
  if (
    head.x < 0 ||
    head.y < 0 ||
    head.x >= tiles ||
    head.y >= tiles
  )
    return true

  return snake.some(p => p.x === head.x && p.y === head.y)
}

function update() {
  direction = nextDirection
  const head = {
    x: snake[0].x + direction.x,
    y: snake[0].y + direction.y
  }

  if (collision(head)) {
    gameOver.value = true
    clearInterval(loop)
    return
  }

  snake.unshift(head)

  // normal food
  if (head.x === food.x && head.y === food.y) {
    score.value++
    spawnParticles(head.x, head.y, '#00fff7')
    triggerShake()
    spawnFood()
  }

  // special food
  else if (
    specialFood &&
    head.x === specialFood.x &&
    head.y === specialFood.y
  ) {
    score.value += 3
    spawnParticles(head.x, head.y, '#ffd700')
    triggerShake()
    specialFood = null
  } else {
    snake.pop()
  }
}

function gameLoop() {
  update()
  draw()
}

/* ===== DRAW ===== */
function drawBackground() {
  ctx.fillStyle = theme.value === 'neon' ? '#050505' : '#111'
  ctx.fillRect(0, 0, canvasSize, canvasSize)

  ctx.strokeStyle = 'rgba(255,255,255,0.05)'
  for (let i = 0; i < tiles; i++) {
    ctx.strokeRect(
      i * tileSize,
      0,
      1,
      canvasSize
    )
    ctx.strokeRect(
      0,
      i * tileSize,
      canvasSize,
      1
    )
  }
}

function drawSnake() {
  snake.forEach((p, i) => {
    ctx.fillStyle = i === 0 ? '#00fff7' : '#00cc7a'
    ctx.fillRect(
      p.x * tileSize,
      p.y * tileSize,
      tileSize,
      tileSize
    )
  })
}

function drawFood() {
  ctx.fillStyle = '#ff4757'
  ctx.fillRect(
    food.x * tileSize,
    food.y * tileSize,
    tileSize,
    tileSize
  )

  if (specialFood) {
    ctx.fillStyle = '#ffd700'
    ctx.fillRect(
      specialFood.x * tileSize,
      specialFood.y * tileSize,
      tileSize,
      tileSize
    )
  }
}

function drawParticles() {
  particles.forEach(p => {
    ctx.fillStyle = p.color
    ctx.fillRect(
      p.x * tileSize,
      p.y * tileSize,
      6,
      6
    )
    p.x += p.vx * 0.1
    p.y += p.vy * 0.1
    p.life--
  })

  particles = particles.filter(p => p.life > 0)
}

function draw() {
  drawBackground()
  drawFood()
  drawSnake()
  drawParticles()
}

/* ===== INPUT ===== */
function handleKey(e) {
  const k = e.key.toLowerCase()

  if ((k === 'arrowup' || k === 'w') && direction.y !== 1)
    nextDirection = { x: 0, y: -1 }
  if ((k === 'arrowdown' || k === 's') && direction.y !== -1)
    nextDirection = { x: 0, y: 1 }
  if ((k === 'arrowleft' || k === 'a') && direction.x !== 1)
    nextDirection = { x: -1, y: 0 }
  if ((k === 'arrowright' || k === 'd') && direction.x !== -1)
    nextDirection = { x: 1, y: 0 }
}

function toggleTheme() {
  theme.value = theme.value === 'neon' ? 'dark' : 'neon'
}

/* ===== WATCH ===== */
watch(score, v => {
  if (v >= 5 && !fury.value) {
    fury.value = true
    speed = furySpeed
    clearInterval(loop)
    loop = setInterval(gameLoop, speed)
  }
})

/* ===== INIT ===== */
onMounted(() => {
  ctx = canvas.value.getContext('2d')
  window.addEventListener('keydown', handleKey)
  resetGame()
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKey)
  clearInterval(loop)
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
  gap: 10px;
  font-family: system-ui;
}

.neon {
  text-shadow: 0 0 8px #00fff7;
}

.top-bar {
  width: 420px;
  display: flex;
  justify-content: space-between;
  font-weight: bold;
}

.canvas-wrap {
  border-radius: 18px;
  overflow: hidden;
}

.canvas-wrap.shake {
  animation: shake 0.12s infinite;
}

canvas {
  border: 2px solid #00fff7;
}

.overlay {
  position: absolute;
  inset: 0;
  background: rgba(0,0,0,0.8);
  display: flex;
  align-items: center;
  justify-content: center;
}

.game-over {
  background: #111;
  padding: 30px;
  border-radius: 16px;
  border: 2px solid #ff4757;
}

button {
  margin-top: 10px;
  padding: 8px 16px;
  border-radius: 10px;
  border: none;
  font-weight: bold;
  cursor: pointer;
}

@keyframes shake {
  0% { transform: translate(1px, 1px); }
  50% { transform: translate(-1px, -1px); }
  100% { transform: translate(1px, -1px); }
}
</style>
