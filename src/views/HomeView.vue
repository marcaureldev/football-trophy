<script setup lang="ts">
import { ref, type Ref } from 'vue'
const WIDTH = 400
const columnWidth = WIDTH / 3

/* Initial position of goalKeeper */
const goalKeeperPosition: Ref<{ x: number; speed: number; move: number }> = ref({
  x: 0,
  speed: 5,
  move: columnWidth / 2,
})

/* Initial position of ball */
const ballPosition: Ref<{ x: number; y: number; speed: number }> = ref({
  x: 0,
  y: 0,
  speed: 3,
})

/* Define keys object to manage which key is selected */
const keys: {
  ArrowRight: boolean
  ArrowLeft: boolean
} = {
  ArrowRight: false,
  ArrowLeft: false,
}

/* Function for starting game */
function startGame() {
  const ball: Element | null = document.querySelector('.ball')
  const goalkeeper: Element | null = document.querySelector('.goalkeeper')

  /* To ensure that ball and goalkeeper are well initialized */
  if (ball && goalkeeper) {
    /* Remove hidden style to the ball and goalKeeper element */
    ball.classList.remove('hidden')
    goalkeeper.classList.remove('hidden')

    /* Choose random column between 0, 1 et 3 */
    const randomColumn = Math.floor(Math.random() * 3)

    /* Set ball position at middle of selected column */
    ballPosition.value.x = randomColumn * columnWidth + columnWidth / 2

    /*Set goalKeeper at middle of the game area */
    goalKeeperPosition.value.x = WIDTH * 0.5

    playGame()
    moveGoalKeeper()
  }
}

function moveGoalKeeper() {
  document.addEventListener('keydown', pressOn)
  document.addEventListener('keyup', pressOff)

  /* Function to start moving when a key is down */
  function pressOn(e: Event) {
    e.preventDefault()
    keys[e.key] = true
    if (keys.ArrowRight && goalKeeperPosition.value.x < 2 * columnWidth) {
      goalKeeperPosition.value.x += columnWidth
    }

    if (keys.ArrowLeft && goalKeeperPosition.value.x > columnWidth) {
      goalKeeperPosition.value.x -= columnWidth
    }
  }

  /* Function called when key is up */
  function pressOff(e: Event) {
    e.preventDefault()
    keys[e.key] = false
  }
}

/* Function to start playing game */
let animationFrame: number;

function playGame() {
  ballPosition.value.y += ballPosition.value.speed
  animationFrame = window.requestAnimationFrame(playGame)
}

function pauseGame() {
  cancelAnimationFrame(animationFrame)
}
</script>

<template>
  <div class="min-h-screen bg-green-950/95 flex">
    <div class="flex-1 p-8 flex items-center justify-center">
      <div class="h-[500px] w-[400px] flex gap-1 rounded-lg relative overflow-hidden">
        <div class="bg-green-950/95 w-1/3 flex justify-center items-end"></div>
        <div class="bg-green-950/95 w-1/3 flex justify-center items-end"></div>
        <div class="bg-green-950/95 w-1/3 flex justify-center items-end"></div>

        <!-- Ball -->
        <div
          class="ball absolute hidden text-white"
          :style="{
            left: `${ballPosition.x}px`,
            top: '-150px',
            transform: `translate(-50%, ${ballPosition.y}px)`,
          }"
        >
          {{ ballPosition.y }}
          <img src="/images/ball.svg" alt="" class="w-8 h-8" />
        </div>

        <!-- Goalkeeper -->
        <div
          class="goalkeeper hidden absolute"
          :style="{
            left: `${goalKeeperPosition.x}px`,
            bottom: 0,
            transform: 'translate(-50%, -50%)',
          }"
        >
          <img src="/images/goal-keeper.svg" alt="" class="w-12 h-12" />
        </div>
      </div>
    </div>

    <div class="w-64 bg-teal-900/80 p-4 flex flex-col gap-4">
      <div class="text-white text-xl mb-4">Score:</div>

      <button
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
        @click="startGame"
      >
        Jouer
      </button>

      <button @click="pauseGame"
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
      >
        Pause
      </button>

      <button
        @click="playGame"
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
      >
        Play
      </button>

      <button
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
      >
        Meilleur Score
      </button>

      <button
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
      >
        Désactiver / Activer le son
      </button>

      <button
        class="w-full bg-green-800 hover:bg-green-700 text-white py-2 px-4 rounded transition-colors"
      >
        Aide
      </button>
    </div>
  </div>
</template>
