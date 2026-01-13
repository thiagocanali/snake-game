<template>
  <div class="game-wrapper" :class="[theme, { shake: gameOver }]">
    <header>
      <h1>🐍 SNAKE OVERDRIVE</h1>

      <div class="scores">
        <span>Score <strong>{{ score }}</strong></span>
        <span>High <strong>{{ highScore }}</strong></span>
      </div>

      <div class="actions">
        <button @click="toggleTheme">
          {{ theme === 'neon' ? '🌙 Dark' : '✨ Neon' }}
        </button>
        <button @click="togglePause">
          {{ paused ? '▶️ Play' : '⏸ Pause' }}
        </button>
      </div>
    </header>

    <canvas ref="canvas" width="400" height="400"></canvas>

    <div class="mobile-controls">
      <button @click="setDir(0, -1)">⬆️</button>
      <div>
        <button @click="setDir(-1, 0)">⬅️</button>
        <button @click="setDir(1, 0)">➡️</button>
      </div>
      <button @click="setDir(0, 1)">⬇️</button>
    </div>

    <div v-if="gameOver" class="overlay">
      <div class="game-over">
        <h2>💀 GAME OVER</h2>
        <p>Score: {{ score }}</p>
        <button @click="resetGame">RESTART</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'

const SIZE = 400
const TILE = 20
const TILES = SIZE / TILE
const BASE_SPEED = 140
const MIN_SPEED = 60

const canvas = ref(null)
let ctx, loop

const score = ref(0)
const highScore = ref(+localStorage.getItem('snake-highscore') || 0)
const theme = ref('neon')
const gameOver = ref(false)
const paused = ref(false)

let speed = BASE_SPEED
let snake = []
let food = {}
let dir = { x: 1, y: 0 }
let nextDir = { x: 1, y: 0 }
let particles = []

function rand() {
  return Math.floor(Math.random() * TILES)
}

function spawnFood() {
  food = { x: rand(), y: rand() }
}

function resetGame() {
  snake = [
    { x: 10, y: 10 },
    { x: 9, y: 10 },
    { x: 8, y: 10 }
  ]
  dir = nextDir = { x: 1, y: 0 }
  score.value = 0
  speed = BASE_SPEED
  gameOver.value = false
  paused.value = false
  particles = []
  spawnFood()
  clearInterval(loop)
  loop = setInterval(gameLoop, speed)
}

function setDir(x, y) {
  if (x === -dir.x && y === -dir.y) return
  nextDir = { x, y }
}

function collision(h) {
  return (
    h.x < 0 || h.y < 0 || h.x >= TILES || h.y >= TILES ||
    snake.some(p => p.x === h.x && p.y === h.y)
  )
}

function explode(x, y) {
  for (let i = 0; i < 15; i++) {
    particles.push({
      x: x * TILE + TILE / 2,
      y: y * TILE + TILE / 2,
      vx: (Math.random() - 0.5) * 4,
      vy: (Math.random() - 0.5) * 4,
      life: 30
    })
  }
}

function update() {
  if (paused.value || gameOver.value) return

  dir = nextDir
  const head = { x: snake[0].x + dir.x, y: snake[0].y + dir.y }

  if (collision(head)) {
    gameOver.value = true
    clearInterval(loop)
    return
  }

  snake.unshift(head)

  if (head.x === food.x && head.y === food.y) {
    score.value++
    explode(food.x, food.y)
    spawnFood()
    speed = Math.max(MIN_SPEED, BASE_SPEED - score.value * 5)
    clearInterval(loop)
    loop = setInterval(gameLoop, speed)
  } else {
    snake.pop()
  }
}

function draw() {
  ctx.fillStyle = theme.value === 'neon' ? '#050505' : '#111'
  ctx.fillRect(0, 0, SIZE, SIZE)

  snake.forEach((p, i) => {
    ctx.fillStyle = i === 0 ? '#00fff7' : '#00cc7a'
    ctx.shadowBlur = 20
    ctx.shadowColor = '#00fff7'
    ctx.fillRect(p.x * TILE, p.y * TILE, TILE, TILE)
  })

  ctx.shadowBlur = 0
  ctx.fillStyle = '#ff4757'
  ctx.fillRect(food.x * TILE, food.y * TILE, TILE, TILE)

  particles.forEach(pt => {
    ctx.fillStyle = 'rgba(0,255,247,.8)'
    ctx.fillRect(pt.x, pt.y, 3, 3)
    pt.x += pt.vx
    pt.y += pt.vy
    pt.life--
  })
  particles = particles.filter(p => p.life > 0)
}

function gameLoop() {
  update()
  draw()
}

function toggleTheme() {
  theme.value = theme.value === 'neon' ? 'dark' : 'neon'
}

function togglePause() {
  paused.value = !paused.value
}

function key(e) {
  const k = e.key.toLowerCase()
  if (k === ' ') togglePause()
  if (k === 'arrowup' || k === 'w') setDir(0, -1)
  if (k === 'arrowdown' || k === 's') setDir(0, 1)
  if (k === 'arrowleft' || k === 'a') setDir(-1, 0)
  if (k === 'arrowright' || k === 'd') setDir(1, 0)
}

watch(score, v => {
  if (v > highScore.value) {
    highScore.value = v
    localStorage.setItem('snake-highscore', v)
  }
})

onMounted(() => {
  ctx = canvas.value.getContext('2d')
  window.addEventListener('keydown', key)
  resetGame()
})
</script>

<style scoped>
.game-wrapper {
  width: 460px;
  padding: 24px;
  border-radius: 24px;
  text-align: center;
  position: relative;
}

.neon {
  background: rgba(0,0,0,.6);
  box-shadow: 0 0 40px #00fff755;
}

.dark {
  background: rgba(0,0,0,.45);
}

canvas {
  border-radius: 16px;
  border: 2px solid #00fff7;
  margin: 10px 0;
}

.scores {
  display: flex;
  justify-content: space-between;
}

.actions {
  display: flex;
  justify-content: space-between;
  margin: 8px 0;
}

button {
  padding: 6px 14px;
  border-radius: 10px;
  border: none;
  cursor: pointer;
  font-weight: bold;
}

.mobile-controls {
  display: none;
}

@media (max-width: 600px) {
  .mobile-controls {
    display: block;
  }
}

.overlay {
  position: absolute;
  inset: 0;
  background: rgba(0,0,0,.8);
  display: flex;
  align-items: center;
  justify-content: center;
}

.game-over {
  padding: 30px;
  border-radius: 16px;
  border: 2px solid #ff4757;
}

.shake {
  animation: shake .4s;
}

@keyframes shake {
  0% { transform: translateX(0); }
  25% { transform: translateX(-6px); }
  50% { transform: translateX(6px); }
  75% { transform: translateX(-6px); }
  100% { transform: translateX(0); }
}
</style>
