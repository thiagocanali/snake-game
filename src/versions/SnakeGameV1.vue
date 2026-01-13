<template>
  <div class="game-wrapper">
    <h1>🐍 Snake Game</h1>

    <canvas
      ref="canvas"
      :width="canvasSize"
      :height="canvasSize"
    ></canvas>

    <p class="hint">
      Use ↑ ↓ ← → ou W A S D
    </p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

/* CONFIGURAÇÕES */
const canvasSize = 400
const tileSize = 20
const tiles = canvasSize / tileSize

const canvas = ref(null)
let ctx = null

/* ESTADO DO JOGO */
let snake = [
  { x: 10, y: 10 },
  { x: 9, y: 10 },
  { x: 8, y: 10 }
]

let direction = { x: 1, y: 0 }
let nextDirection = { x: 1, y: 0 }

/* DESENHO */
function drawBackground() {
  ctx.fillStyle = '#111'
  ctx.fillRect(0, 0, canvasSize, canvasSize)

  ctx.strokeStyle = '#1e1e1e'
  for (let i = 0; i < tiles; i++) {
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

/* LÓGICA */
function update() {
  direction = nextDirection

  const head = {
    x: snake[0].x + direction.x,
    y: snake[0].y + direction.y
  }

  snake.unshift(head)
  snake.pop()
}

function gameLoop() {
  drawBackground()
  update()
  drawSnake()
}

/* CONTROLES */
function handleKey(e) {
  const key = e.key.toLowerCase()

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

  setInterval(gameLoop, 120)
})
</script>

<style scoped>
.game-wrapper {
  background: rgba(0, 0, 0, 0.45);
  padding: 30px 40px;
  border-radius: 20px;
  text-align: center;
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.5);
}

h1 {
  margin-bottom: 15px;
}

canvas {
  background: #111;
  border-radius: 12px;
  border: 2px solid #00ff9c;
}

.hint {
  margin-top: 12px;
  font-size: 0.9rem;
  opacity: 0.7;
}
</style>