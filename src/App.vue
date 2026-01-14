<template>
  <div class="min-h-screen bg-gradient-to-br from-pink-600 to-slate-950 flex flex-col items-center justify-center p-4">
      <h1 class="text-5xl font-bold mb-8 text-white">Memory Card Game</h1>
      <p class="text-xl text-white">Match pairs of cards to win!</p>

      <!-- Difficulty selector -->
       <div class="bg-white rounded-lg px-6 py-3 mb-4 shadow-lg">
        <p class="text-sm font-semibold text-gray-600 mb-2 text-center">Difficulty</p>
        <div class="flex gap-2">
          <button 
            @click="changeDifficulty('easy')"
            :class="['px-4 py-2 rounded-lg font-semibold transition-all',
              difficulty === 'easy'
                ? 'bg-green-500 text-white'
                : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
            ]">Easy</button>
            <button
              @click="changeDifficulty('medium')"
              :class="['px-4 py-2 rounded-lg font-semibold transition-all',
                difficulty === 'medium'
                  ? 'bg-yellow-500 text-white'
                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
              ]">Medium</button>
              <button
              @click="changeDifficulty('hard')"
              :class="['px-4 py-2 rounded-lg font-semibold transition-all',
                difficulty === 'hard'
                  ? 'bg-red-500 text-white'
                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
              ]">Hard</button>
        </div>
       </div>

      <!-- Game stats -->
       <div class="bg-white rounded-lg px-6 py-3 mb-6 shadow-lg">
        <p class="text-lg font-semibold text-gray-700">Moves: {{ moves }}</p>
        <p class="text-lg font-semibold text-gray-700">Time: {{ formattedTime }}</p>
       </div>

       <!-- Win Message -->
        <div v-if="isGameWon" class="bg-green-500 text-white px-8 py-4 rounded-lg mb-4 shadow-lg animate-bounce">
          <p class="text-2xl font-bold">🎊 You Won! 🎊</p>
          <p class="text-lg">Moves: {{ moves }} | Time: {{ formattedTime }}</p>
        </div>
        <!-- Confetti animation -->
         <Confetti v-if="isGameWon" />

      <!-- Grid of cards - dynamic columns -->
       <div class="grid gap-4 mb-6" :style="{display: 'grid', gridTemplateColumns: `repeat(${gridCols}, 1fr)`, gap: '1rem'}">
        <Card
           v-for="card in cards"
           :key="card.id"
           :id="card.id"
           :emoji="card.emoji"
           :isFlipped="card.isFlipped || card.isMatched"
           :isShaking="card.isShaking"
           @flip="handleFlip"
           />
       </div>

       <!-- new game Button -->
        <button @click="newGame" class="bg-white text-purple-700 font-bold py-3 px-8 rounded-lg shadow-lg hover: bg-gray-100 hover:scale-105 transition-all duration-200">
          New Game
        </button>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onUnmounted } from 'vue'
import Card from './components/Card.vue'
import Confetti from './components/Confetti.vue'

//Typescript interface for my card data
interface CardType {
  id: number
  emoji: string
  pairId: number // to match
  isFlipped: boolean
  isMatched: boolean
  isShaking: boolean
}

// Game state
const cards = ref<CardType[]>([])
const flippedCards = ref<CardType[]>([])
const moves = ref(0)
const isChecking = ref(false)
const timer = ref(0)
const timerInterval = ref<ReturnType<typeof setInterval> | null>(null)
  type Difficulty = 'easy' | 'medium' |'hard'
  const difficulty = ref<Difficulty>('medium')

//Emojis on card
const allEmojis = ['☀️', '🌙', '⭐', '🍀', '💎', '🍒', '🍄‍🟫', '💥', '🐦‍🔥', '👾', '🪼', '🖤']

// Computed property: checking if u won
const isGameWon = computed(() => {
  return cards.value.length > 0 && cards.value.every(card => card.isMatched)
})

//fomat time to mm:ss
  const formattedTime = computed(() => {
    const minutes = Math.floor(timer.value / 60)
    const seconds = timer.value % 60
    return `${minutes}:${seconds.toString().padStart(2, '0')}`
  })

  // start timer
  const startTimer = (): void => {
    if (timerInterval.value) {
      clearInterval(timerInterval.value)
    }
    timer.value = 0
    timerInterval.value = setInterval(() => {
      timer.value++
    }, 1000)
  }

  //stop timer
  const stopTimer = (): void => {
    if (timerInterval.value) {
      clearInterval(timerInterval.value)
      timerInterval.value = null
    }
  }

  //get grid size based on difficulty
  const getGridConfig = (diff: Difficulty) => {
    switch (diff) {
      case 'easy':
        return { pairs: 6, cols: 4, rows: 3 }
      case 'medium':
        return { pairs: 8, cols: 4, rows: 4 }
      case 'hard':
        return { pairs: 12, cols: 6, rows: 4 }
    }
  }

  // computed property for grid columns
  const gridCols = computed(() => getGridConfig(difficulty.value).cols)

// Initialize/Shuffle
const InitializeGame = (): void => {
  const cardPairs: CardType[] = []
  const config = getGridConfig(difficulty.value)
  const emojisToUse: string[] = allEmojis.slice(0, config.pairs)

  // Create pairs
  emojisToUse.forEach((emoji: string, index: number) => {
    cardPairs.push(
      { id: index * 2, emoji, pairId: index , isFlipped: false, isMatched: false, isShaking: false },
      { id: index * 2 + 1, emoji, pairId: index, isFlipped: false, isMatched: false, isShaking: false }
    )
  })

// Shuffle cards (Fisher-Yates shuffle algorithm
for (let i = cardPairs.length - 1; i > 0; i--) {
  const j = Math.floor(Math.random() * (i + 1))
  const temp = cardPairs[i]
  cardPairs[i] = cardPairs[j]
  cardPairs[j] = temp
}

cards.value = cardPairs
flippedCards.value = []
moves.value = 0
timer.value = 0
}

//Handle card flip
const handleFlip = (id: number): void => {
  //prevent flips while checking for matches
  if (isChecking.value) return

  //prevent more than 2 card flips at a time
  if (flippedCards.value.length >= 2) return

  const card = cards.value.find((c: CardType) => c.id === id)
  if (!card || card.isMatched || card.isFlipped) return

  // start timer on first flip
  if (timer.value === 0 && !timerInterval.value) {
    startTimer()
  }

  // flip card
  card.isFlipped = true
  flippedCards.value.push(card)

  // check for matches hwen 2 cards r flipped
  if (flippedCards.value.length === 2) {
    moves.value++
    checkForMatch()
  }
}

// check if 2 matching
const checkForMatch = (): void => {
  isChecking.value = true
  const [card1, card2] = flippedCards.value

  if (!card1 || !card2) {
    isChecking.value = false
    return
  }

  if (card1.pairId === card2.pairId) {
    //match foundd
    card1.isMatched = true
    card2.isMatched = true
    flippedCards.value = []
    isChecking.value = false
    //check if u won and stop timer
    if (cards.value.every(card => card.isMatched)) {
      stopTimer()
    }
  } else {
    //no match - flips back after shake
    card1.isShaking = true
    card2.isShaking = true

    setTimeout(() => {
      card1.isShaking = false
      card2.isShaking = false
      card1.isFlipped = false
      card2.isFlipped = false
      flippedCards.value = []
      isChecking.value = false
    }, 1000)
  }
}

//change difficulty
const changeDifficulty = (newDifficulty: Difficulty): void => {
  difficulty.value = newDifficulty
  stopTimer()
  InitializeGame()
}

// restart
const newGame = (): void => {
  stopTimer()
  InitializeGame()
}

//cleanup timer when component unmounts
onUnmounted(() => {
  stopTimer()
})

//initialize on mount
InitializeGame()
</script>