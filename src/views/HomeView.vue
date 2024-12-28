<template>
  <div class="min-h-screen bg-green-950/95 flex">
    <div class="flex-1 p-8 flex items-center justify-center">
      <div class="h-[500px] w-[400px] flex gap-1 rounded-lg relative">
        <div class="bg-green-950/95 w-full flex justify-center items-end"></div>
        <div class="bg-green-950/95 w-full flex justify-center items-end">
          <!-- Ball -->
          <div
            ref="ballRef"
            class="absolute top-0"
            :style="{ transform: `translate(${ballPosition.x}px, ${ballPosition.y}px)` }"
          >
            <img src="/images/ball.svg " alt="" />
          </div>
          <!-- Goalkeeper -->
          <div ref="goalkeeperRef" class="flex justify-center" :style="{ transform: `translateX(${goalkeeper.x}px)` }">
            <img src="/images/goal-keeper.svg" alt="" class="w-12 h-12" />
          </div>
        </div>
        <div class="bg-green-950/95 w-full flex justify-center items-end"></div>
      </div>
    </div>

    <div class="w-64 bg-teal-900/80 p-4 flex flex-col gap-4">
      <div class="text-white text-xl mb-4">Score : {{ score }}</div>

      <button
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
        @click="startGame"
        :disabled="gameStarted"
      >
        Jouer
      </button>

      <button
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
        @click="pauseGame"
      >
        Pause
      </button>

      <button
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
        @click="resumeGame"
      >
        Play
      </button>

      <button
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
      >
        Meilleur Score: {{ bestScore }}
      </button>

      <button
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
        @click="toggleSound"
      >
        {{ soundEnabled ? 'Désactiver' : 'Activer' }} le son
      </button>

      <button
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
      >
        Aide
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

// Game state
const gameStarted = ref(false)
const gamePaused = ref(false)
const score = ref(0)
const bestScore = ref(0)
const soundEnabled = ref(true)

// Elements refs
const ballRef = ref<HTMLElement | null>(null)
const goalkeeperRef = ref<HTMLElement | null>(null)


const ballPosition = ref({ x: 4, y: -50 })
const ballSpeed = ref({ x: 0, y: 2 })


const goalkeeper = ref({
  x: 0,
  speed: 15,
  column: 1, 
})

// Game loop
let animationFrame: number

function startGame() {
  if (gameStarted.value) return

  gameStarted.value = true
  gamePaused.value = false
  score.value = 0
  resetBall()
  gameLoop()
}

function pauseGame() {
  gamePaused.value = true
  cancelAnimationFrame(animationFrame)
}

function resumeGame() {
  if (!gameStarted.value) return
  gamePaused.value = false
  gameLoop()
}

function resetBall() {
  // Random x position at the top
  ballPosition.value = {
    x: Math.random() * 400 + 50,
    y: 0,
  }
}

function checkCollision(): boolean {
  if (!ballRef.value || !goalkeeperRef.value) return false

  const ballRect = ballRef.value.getBoundingClientRect()
  const goalkeeperRect = goalkeeperRef.value.getBoundingClientRect()

  return !(
    ballRect.bottom < goalkeeperRect.top ||
    ballRect.top > goalkeeperRect.bottom ||
    ballRect.right < goalkeeperRect.left ||
    ballRect.left > goalkeeperRect.right
  )
}

function gameLoop() {
  if (gamePaused.value) return

  // Move ball
  ballPosition.value.y += ballSpeed.value.y

  // Check collision
  if (checkCollision()) {
    score.value++
    if (score.value > bestScore.value) {
      bestScore.value = score.value
    }
    resetBall()
  }

  // Check if ball passed goalkeeper
  if (ballPosition.value.y > 500) {
    gameOver()
    return
  }

  animationFrame = requestAnimationFrame(gameLoop)
}

function gameOver() {
  gameStarted.value = false
  alert(`Game Over! Score: ${score.value}`)
}

function moveGoalkeeper(direction: 'left' | 'right') {
  const columnWidth = 166 // Approximate width of each column

  if (direction === 'left' && goalkeeper.value.column > 0) {
    goalkeeper.value.column--
    goalkeeper.value.x -= columnWidth
  } else if (direction === 'right' && goalkeeper.value.column < 2) {
    goalkeeper.value.column++
    goalkeeper.value.x += columnWidth
  }
}

function handleKeydown(event: KeyboardEvent) {
  if (!gameStarted.value || gamePaused.value) return

  switch (event.key) {
    case 'ArrowLeft':
      moveGoalkeeper('left')
      break
    case 'ArrowRight':
      moveGoalkeeper('right')
      break
  }
}

function toggleSound() {
  soundEnabled.value = !soundEnabled.value
}

// Event listeners
onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
  cancelAnimationFrame(animationFrame)
})
</script>
