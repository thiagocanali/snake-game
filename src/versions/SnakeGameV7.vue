<template>
  <div class="v7">
    <h1>🐍 Snake V7 — APOCALYPSE</h1>

    <p>
      Esta versão ainda não foi domada.
    </p>

    <p class="warn">
      ⚠️ Aqui o caos será absoluto.
    </p>

    <button @click="explode">
      NÃO CLIQUE 😈
    </button>

    <p v-if="boom" class="boom">
      💥 BOOOOOOM 💥
    </p>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const boom = ref(false)

function explode() {
  boom.value = true
}
</script>

<style scoped>
.v7 {
  padding: 40px;
  text-align: center;
  border-radius: 20px;
  background: radial-gradient(circle, #220000, #000);
  color: #ff4d4d;
  box-shadow: 0 0 40px #ff000055;
}

.warn {
  margin-top: 10px;
  opacity: 0.8;
}

button {
  margin-top: 20px;
  padding: 12px 20px;
  border-radius: 14px;
  border: none;
  cursor: pointer;
  background: #ff0000;
  color: white;
  font-weight: bold;
}

.boom {
  margin-top: 20px;
  font-size: 1.5rem;
  animation: shake 0.4s infinite;
}

@keyframes shake {
  0% { transform: translateX(-2px); }
  50% { transform: translateX(2px); }
  100% { transform: translateX(-2px); }
}
</style>
