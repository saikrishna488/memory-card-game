<template>
  <div class="min-h-screen bg-gradient-to-br from-pink-600 to-slate-950 flex flex-col items-center justify-center p-4">
      <h1 class="text-5xl font-bold mb-8 text-white">Memory Card Game</h1>
      <p class="text-xl text-white">Match pairs of cards to win!</p>

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

      <!-- Grid of cards -->
       <div class="gri grid-cols-4 gap-4 mb-6" style="display: grid; grid-template-columns: repeat(4, 1fr); gap: 1rem;">
        <Card
           v-for="card in cards"
           :key="card.id"
           :id="card.id"
           :emoji="card.emoji"
           :isFlipped="card.isFlipped || card.isMatched"
           @flip="handleFlip"
           />
       </div>

       <!-- Restart Button -->
        <button @click="restartGame" class="bg-white text-purple-700 font-bold py-8 rounded-lg shadow-lg hover: bg-gray-100 transition-colors">
          Restart Game
        </button>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onUnmounted } from 'vue'
import Card from './components/Card.vue'

//Typescript interface for my card data
interface CardType {
  id: number
  emoji: string
  pairId: number // to match
  isFlipped: boolean
  isMatched: boolean
}

// Game state
const cards = ref<CardType[]>([])
const flippedCards = ref<CardType[]>([])
const moves = ref(0)
const isChecking = ref(false)
const timer = ref(0)
const timerInterval = ref<ReturnType<typeof setInterval> | null>(null)

//Emojis on card
const emojis = ['☀️', '🌙', '⭐', '🍀', '💎', '🍒', '🍄‍🟫', '💥']

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

// Initialize/Shuffle
const InitializeGame = () => {
  const cardPairs: CardType[] = []

  // Create pairs
  emojis.forEach((emoji, index) => {
    cardPairs.push(
      { id: index * 2, emoji, pairId: index , isFlipped: false, isMatched: false },
      { id: index * 2 + 1, emoji, pairId: index, isFlipped: false, isMatched: false }
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
    //no match - flips back
    setTimeout(() => {
      card1.isFlipped = false
      card2.isFlipped = false
      flippedCards.value = []
      isChecking.value = false
    }, 1000)
  }
}

// restart
const restartGame = (): void => {
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