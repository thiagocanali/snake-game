<template>
  <div class="wrapper" :class="[theme, { shake: rage }]">
    <header>
      <h1>🐍 SNAKE APOCALYPSE</h1>

      <div class="hud">
        <span>Score <b>{{ score }}</b></span>
        <span>High <b>{{ highScore }}</b></span>
        <span>Combo <b>x{{ combo }}</b></span>
      </div>

      <div class="buttons">
        <button @click="toggleTheme">
          {{ theme === 'neon' ? '🌙 Dark' : '✨ Neon' }}
        </button>
        <button @click="togglePause">
          {{ paused ? '▶️ Play' : '⏸ Pause' }}
        </button>
      </div>
    </header>

    <canvas ref="canvas" width="400" height="400"></canvas>

    <p class="status">
      {{ rage ? '🔥 RAGE MODE ATIVO 🔥' : '🎮 Jogue para ativar o caos' }}
    </p>

    <div v-if="gameOver" class="overlay">
      <div class="game-over">
        <h2>💀 FIM DO APOCALIPSE</h2>
        <p>Score: {{ score }}</p>
        <button @click="resetGame">REINICIAR</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'

/* CONFIG */
const SIZE = 400
const TILE = 20
const GRID = SIZE / TILE
const BASE_SPEED = 140
const MIN_SPEED = 55

/* REFS */
const canvas = ref(null)
let ctx, loop

/* STATE */
const score = ref(0)
const combo = ref(1)
const highScore = ref(+localStorage.getItem('snake-highscore') || 0)
const theme = ref('neon')
const gameOver = ref(false)
const paused = ref(false)
const rage = ref(false)

let speed = BASE_SPEED
let snake = []
let food = {}
let obstacles = []
let dir = { x: 1, y: 0 }
let nextDir = { x: 1, y: 0 }

/* HELPERS */
const rand = () => Math.floor(Math.random() * GRID)

function spawnFood() {
  food = {
    x: rand(),
    y: rand(),
    type: Math.random() < 0.2 ? 'boost' : Math.random() < 0.1 ? 'slow' : 'normal'
  }
}

function spawnObstacles() {
  obstacles = []
  for (let i = 0; i < Math.min(5, score.value); i++) {
    obstacles.push({ x: rand(), y: rand() })
  }
}

/* GAME */
function resetGame() {
  snake = [
    { x: 10, y: 10 },
    { x: 9, y: 10 },
    { x: 8, y: 10 }
  ]
  dir = nextDir = { x: 1, y: 0 }
  score.value = 0
  combo.value = 1
  rage.value = false
  speed = BASE_SPEED
  gameOver.value = false
  paused.value = false
  spawnFood()
  spawnObstacles()
  clearInterval(loop)
  loop = setInterval(gameLoop, speed)
}

function collision(h) {
  return (
    h.x < 0 || h.y < 0 || h.x >= GRID || h.y >= GRID ||
    snake.some(p => p.x === h.x && p.y === h.y) ||
    obstacles.some(o => o.x === h.x && o.y === h.y)
  )
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
    score.value += combo.value
    combo.value++
    if (combo.value >= 5) rage.value = true

    if (food.type === 'boost') speed = Math.max(MIN_SPEED, speed - 10)
    if (food.type === 'slow') speed += 15

    spawnFood()
    spawnObstacles()
    clearInterval(loop)
    loop = setInterval(gameLoop, speed)
  } else {
    combo.value = 1
    rage.value = false
    snake.pop()
  }
}

/* DRAW */
function draw() {
  ctx.fillStyle = theme.value === 'neon' ? '#040404' : '#111'
  ctx.fillRect(0, 0, SIZE, SIZE)

  obstacles.forEach(o => {
    ctx.fillStyle = '#ff4757'
    ctx.fillRect(o.x * TILE, o.y * TILE, TILE, TILE)
  })

  snake.forEach((p, i) => {
    ctx.fillStyle = rage.value
      ? '#ff9f1a'
      : i === 0
      ? '#00fff7'
      : '#00cc7a'
    ctx.shadowBlur = rage.value ? 25 : 15
    ctx.shadowColor = ctx.fillStyle
    ctx.fillRect(p.x * TILE, p.y * TILE, TILE, TILE)
  })

  ctx.shadowBlur = 0
  ctx.fillStyle =
    food.type === 'boost'
      ? '#1dd1a1'
      : food.type === 'slow'
      ? '#54a0ff'
      : '#ff6b6b'

  ctx.fillRect(food.x * TILE, food.y * TILE, TILE, TILE)
}

function gameLoop() {
  update()
  draw()
}

/* INPUT */
function setDir(x, y) {
  if (x === -dir.x && y === -dir.y) return
  nextDir = { x, y }
}

function key(e) {
  const k = e.key.toLowerCase()
  if (k === ' ') togglePause()
  if (k === 'arrowup' || k === 'w') setDir(0, -1)
  if (k === 'arrowdown' || k === 's') setDir(0, 1)
  if (k === 'arrowleft' || k === 'a') setDir(-1, 0)
  if (k === 'arrowright' || k === 'd') setDir(1, 0)
}

function toggleTheme() {
  theme.value = theme.value === 'neon' ? 'dark' : 'neon'
}

function togglePause() {
  paused.value = !paused.value
}

/* WATCH */
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
.wrapper {
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

.hud {
  display: flex;
  justify-content: space-between;
  font-size: .9rem;
}

.buttons {
  display: flex;
  justify-content: space-between;
  margin: 6px 0;
}

.status {
  font-size: .8rem;
  opacity: .8;
}

button {
  padding: 6px 14px;
  border-radius: 10px;
  border: none;
  font-weight: bold;
  cursor: pointer;
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
  background: #111;
  padding: 30px;
  border-radius: 16px;
  border: 2px solid #ff4757;
}

.shake {
  animation: shake .25s infinite;
}

@keyframes shake {
  0% { transform: translate(0); }
  25% { transform: translate(-4px, 2px); }
  50% { transform: translate(4px, -2px); }
  75% { transform: translate(-4px, -2px); }
  100% { transform: translate(0); }
}
</style>
