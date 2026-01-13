<template>
  <div class="game-wrapper">
    <h1>🐍 Snake Game</h1>

    <div class="score">
      Score: <strong>{{ score }}</strong>
    </div>

    <canvas
      ref="canvas"
      :width="canvasSize"
      :height="canvasSize"
    ></canvas>

    <p class="hint">
      Use ↑ ↓ ← → ou W A S D — pressione ESPAÇO para reiniciar
    </p>

    <div v-if="gameOver" class="overlay">
      <div class="game-over">
        <h2>💀 Game Over</h2>
        <p>Score final: {{ score }}</p>
        <button @click="resetGame">Jogar novamente</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

/* CONFIG */
const canvasSize = 400
const tileSize = 20
const tiles = canvasSize / tileSize
const speed = 120

const canvas = ref(null)
let ctx = null
let loop = null

/* ESTADO */
const score = ref(0)
const gameOver = ref(false)

let snake = []
let food = {}
let direction = { x: 1, y: 0 }
let nextDirection = { x: 1, y: 0 }

/* FUNÇÕES */
function randomPosition() {
  return {
    x: Math.floor(Math.random() * tiles),
    y: Math.floor(Math.random() * tiles)
  }
}

function spawnFood() {
  food = randomPosition()
}

function resetGame() {
  snake = [
    { x: 10, y: 10 },
    { x: 9, y: 10 },
    { x: 8, y: 10 }
  ]
  direction = { x: 1, y: 0 }
  nextDirection = { x: 1, y: 0 }
  score.value = 0
  gameOver.value = false
  spawnFood()

  clearInterval(loop)
  loop = setInterval(gameLoop, speed)
}

/* DESENHO */
function drawBackground() {
  ctx.fillStyle = '#111'
  ctx.fillRect(0, 0, canvasSize, canvasSize)
}

function drawSnake() {
  snake.forEach((part, index) => {
    ctx.fillStyle = index === 0 ? '#00ff9c' : '#00cc7a'
    ctx.fillRect(
      part.x * tileSize,
      part.y * tileSize,
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

/* LÓGICA */
function checkCollision(head) {
  // parede
  if (
    head.x < 0 ||
    head.y < 0 ||
    head.x >= tiles ||
    head.y >= tiles
  ) {
    return true
  }

  // corpo
  return snake.some(
    (part) => part.x === head.x && part.y === head.y
  )
}

function update() {
  if (gameOver.value) return

  direction = nextDirection

  const head = {
    x: snake[0].x + direction.x,
    y: snake[0].y + direction.y
  }

  if (checkCollision(head)) {
    gameOver.value = true
    clearInterval(loop)
    return
  }

  snake.unshift(head)

  // comeu comida
  if (head.x === food.x && head.y === food.y) {
    score.value++
    spawnFood()
  } else {
    snake.pop()
  }
}

function gameLoop() {
  drawBackground()
  update()
  drawFood()
  drawSnake()
}

/* CONTROLES */
function handleKey(e) {
  const key = e.key.toLowerCase()

  if (key === ' ' && gameOver.value) {
    resetGame()
  }

  if ((key === 'arrowup' || key === 'w') && direction.y !== 1)
    nextDirection = { x: 0, y: -1 }

  if ((key === 'arrowdown' || key === 's') && direction.y !== -1)
    nextDirection = { x: 0, y: 1 }

  if ((key === 'arrowleft' || key === 'a') && direction.x !== 1)
    nextDirection = { x: -1, y: 0 }

  if ((key === 'arrowright' || key === 'd') && direction.x !== -1)
    nextDirection = { x: 1, y: 0 }
}

onMounted(() => {
  ctx = canvas.value.getContext('2d')
  window.addEventListener('keydown', handleKey)
  resetGame()
})
</script>

<style scoped>
.game-wrapper {
  background: rgba(0, 0, 0, 0.45);
  padding: 30px 40px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.5);
  position: relative;
}

.score {
  margin-bottom: 10px;
  font-size: 1.1rem;
}

canvas {
  background: #111;
  border-radius: 12px;
  border: 2px solid #00ff9c;
}

.hint {
  margin-top: 12px;
  font-size: 0.85rem;
  opacity: 0.7;
}

/* GAME OVER */
.overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.7);
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.game-over {
  background: #111;
  padding: 30px 40px;
  border-radius: 16px;
  border: 2px solid #ff4757;
}

.game-over h2 {
  margin-bottom: 10px;
}

button {
  margin-top: 15px;
  padding: 10px 20px;
  background: #00ff9c;
  border: none;
  border-radius: 10px;
  font-weight: bold;
  cursor: pointer;
}

button:hover {
  opacity: 0.9;
}
</style>