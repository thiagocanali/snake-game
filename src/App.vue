<template>
  <div class="app">
    <!-- MENU -->
    <div v-if="!current" class="menu">
      <h1>🐍 Snake Game</h1>
      <p class="subtitle">Escolha a versão do caos</p>

      <div class="versions">
        <button
          v-for="v in versions"
          :key="v.key"
          @click="load(v.key)"
        >
          {{ v.name }}
        </button>
      </div>
    </div>

    <!-- GAME -->
    <div v-else class="game-wrapper">
      <button class="back" @click="current = null">
        ⬅ Voltar ao menu
      </button>

      <component :is="currentComponent" />
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue'

import EmBreve from './versions/EmBreve.vue'
import SnakeGameV1 from './versions/SnakeGamev1.vue'
import SnakeGameV2 from './versions/SnakeGameV2.vue'
import SnakeGameV3 from './versions/SnakeGameV3.vue'
import SnakeGameV4 from './versions/SnakeGameV4.vue'
import SnakeGameV5 from './versions/SnakeGameV5.vue'
import SnakeGameV6 from './versions/SnakeGameV6.vue'
import SnakeGameV7 from './versions/SnakeGameV7.vue'

const current = ref(null)

const versions = [
  { key: 'v1', name: 'Snake V1 🟢' },
  { key: 'v2', name: 'Snake V2 🔵' },
  { key: 'v3', name: 'Snake V3 🟣' },
  { key: 'v4', name: 'Snake V4 🟠' },
  { key: 'v5', name: 'Snake V5 🔥' },
  { key: 'v6', name: 'Snake V6 💀' },
  { key: 'v7', name: 'Snake V7 APOCALYPSE 😈' },
  { key: 'soon', name: 'Em breve 🚧' }
]

const map = {
  v1: SnakeGameV1,
  v2: SnakeGameV2,
  v3: SnakeGameV3,
  v4: SnakeGameV4,
  v5: SnakeGameV5,
  v6: SnakeGameV6,
  v7: SnakeGameV7,
  soon: EmBreve
}

function load(key) {
  current.value = key
}

const currentComponent = computed(() => map[current.value])
</script>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.menu {
  text-align: center;
}

.subtitle {
  opacity: .7;
  margin-bottom: 20px;
}

.versions {
  display: grid;
  gap: 12px;
}

button {
  padding: 14px 20px;
  border-radius: 14px;
  border: none;
  font-weight: bold;
  cursor: pointer;
  background: linear-gradient(135deg, #00fff7, #00cc7a);
  color: #000;
}

.back {
  margin-bottom: 12px;
  background: #222;
  color: #fff;
}

.game-wrapper {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>
