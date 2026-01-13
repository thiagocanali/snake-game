<template>
  <div class="game-wrapper" :class="theme">
    <header>
      <h1>🐍 Snake Game</h1>

      <div class="scores">
        <span>Score: <strong>{{ score }}</strong></span>
        <span>High: <strong>{{ highScore }}</strong></span>
      </div>

      <button class="theme-btn" @click="toggleTheme">
        {{ theme === 'neon' ? '🌙 Dark' : '✨ Neon' }}
      </button>
    </header>

    <canvas
      ref="canvas"
      :width="canvasSize"
      :height="canvasSize"
    ></canvas>

    <p class="hint">
      ↑ ↓ ← → / WASD — Mobile: arraste — Espaço reinicia
    </p>

    <div v-if="gameOver" class="overlay">
      <div class="game-over">
        <h2>💀 Game Over</h2>
        <p>Score: {{ score }}</p>
        <button @click="resetGame">Jogar novamente</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'

/* CONFIG */
const canvasSize = 400
const tileSize = 20
const tiles = canvasSize / tileSize
const baseSpeed = 140
const minSpeed = 60

/* REFS */
const canvas = ref(null)
let ctx
let loop

/* STATE */
const score = ref(0)
const highScore = ref(
  Number(localStorage.getItem('snake-highscore')) || 0
)
const gameOver = ref(false)
const theme = ref('neon')

let speed = baseSpeed
let snake = []
let food = {}
let direction = { x: 1, y: 0 }
let nextDirection = { x: 1, y: 0 }

/* HELPERS */
function randomPos() {
  return {
    x: Math.floor(Math.random() * tiles),
    y: Math.floor(Math.random() * tiles)
  }
}

function spawnFood() {
  food = randomPos()
}

/* GAME */
function resetGame() {
  snake = [
    { x: 10, y: 10 },
    { x: 9, y: 10 },
    { x: 8, y: 10 }
  ]
  direction = { x: 1, y: 0 }
  nextDirection = { x: 1, y: 0 }
  score.value = 0
  speed = baseSpeed
  gameOver.value = false
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

  return snake.some(
    (p) => p.x === head.x && p.y === head.y
  )
}

function update() {
  if (gameOver.value) return

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

  if (head.x === food.x && head.y === food.y) {
    score.value++
    spawnFood()

    speed = Math.max(minSpeed, baseSpeed - score.value * 5)
    clearInterval(loop)
    loop = setInterval(gameLoop, speed)
  } else {
    snake.pop()
  }
}

/* DRAW */
function drawBackground() {
  ctx.fillStyle = theme.value === 'neon' ? '#050505' : '#111'
  ctx.fillRect(0, 0, canvasSize, canvasSize)
}

function drawSnake() {
  snake.forEach((p, i) => {
    ctx.fillStyle =
      i === 0
        ? theme.value === 'neon'
          ? '#00fff7'
          : '#00ff9c'
        : '#00cc7a'

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
}

function gameLoop() {
  drawBackground()
  update()
  drawFood()
  drawSnake()
}

/* INPUT */
function handleKey(e) {
  const k = e.key.toLowerCase()

  if (k === ' ' && gameOver.value) resetGame()

  if ((k === 'arrowup' || k === 'w') && direction.y !== 1)
    nextDirection = { x: 0, y: -1 }
  if ((k === 'arrowdown' || k === 's') && direction.y !== -1)
    nextDirection = { x: 0, y: 1 }
  if ((k === 'arrowleft' || k === 'a') && direction.x !== 1)
    nextDirection = { x: -1, y: 0 }
  if ((k === 'arrowright' || k === 'd') && direction.x !== -1)
    nextDirection = { x: 1, y: 0 }
}

/* MOBILE SWIPE */
let startX = 0
let startY = 0

function touchStart(e) {
  startX = e.touches[0].clientX
  startY = e.touches[0].clientY
}

function touchEnd(e) {
  const dx = e.changedTouches[0].clientX - startX
  const dy = e.changedTouches[0].clientY - startY

  if (Math.abs(dx) > Math.abs(dy)) {
    if (dx > 0 && direction.x !== -1) nextDirection = { x: 1, y: 0 }
    if (dx < 0 && direction.x !== 1) nextDirection = { x: -1, y: 0 }
  } else {
    if (dy > 0 && direction.y !== -1) nextDirection = { x: 0, y: 1 }
    if (dy < 0 && direction.y !== 1) nextDirection = { x: 0, y: -1 }
  }
}

/* WATCHERS */
watch(score, (v) => {
  if (v > highScore.value) {
    highScore.value = v
    localStorage.setItem('snake-highscore', v)
  }
})

function toggleTheme() {
  theme.value = theme.value === 'neon' ? 'dark' : 'neon'
}

/* INIT */
onMounted(() => {
  ctx = canvas.value.getContext('2d')
  window.addEventListener('keydown', handleKey)
  canvas.value.addEventListener('touchstart', touchStart)
  canvas.value.addEventListener('touchend', touchEnd)
  resetGame()
})
</script>

<style scoped>
.game-wrapper {
  padding: 25px 30px;
  border-radius: 22px;
  text-align: center;
  position: relative;
  width: 460px;
}

.neon {
  background: rgba(0, 0, 0, 0.6);
  box-shadow: 0 0 40px #00fff755;
}

.dark {
  background: rgba(0, 0, 0, 0.45);
}

header {
  margin-bottom: 10px;
}

.scores {
  display: flex;
  justify-content: space-between;
  margin: 10px 0;
}

canvas {
  border-radius: 14px;
  border: 2px solid #00fff7;
  touch-action: none;
}

.theme-btn {
  margin-top: 5px;
  padding: 6px 14px;
  border-radius: 10px;
  border: none;
  cursor: pointer;
  font-weight: bold;
}

.hint {
  margin-top: 8px;
  font-size: 0.8rem;
  opacity: 0.7;
}

/* GAME OVER */
.overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.75);
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 22px;
}

.game-over {
  background: #111;
  padding: 30px;
  border-radius: 16px;
  border: 2px solid #ff4757;
}
</style>