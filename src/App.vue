<template>
  <div class="min-h-screen bg-gradient-to-br from-pink-600 to-slate-950 flex items-center justify-center p-8">
    <!-- Main container with sidebar layout -->
     <div class="flex gap-8 w-full max-w-7xl">

      <!-- left sidebar - Stats & controls -->
       <div class="flex flex-col gap-4 w-80">
        <!-- Title -->
         <div class="bg-black rounded-lg p-6 shadow-lg">
          <h1 class="text-3xl font-bold text-violet-200 text-center">Memory Game</h1>
         </div>

         <!-- Difficulty selector -->
          <div class="bg-white rounded-lg p-6 shadow-lg">
            <p class="text-sm font-semibold text-gray-600 mb-3">Difficulty</p>
            <div class="flex flex-col gap-2">
              <button
                @click="changeDifficulty('easy')"
                :class="[
                  'px-4 py-2 rounded-lg font-semibold transition-all',
                  difficulty === 'easy'
                  ? 'bg-green-500 text-white'
                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
                ]">Easy (3x4)</button>
                <button
                @click="changeDifficulty('medium')"
                :class="[
                  'px-4 py-2 rounded-lg font-semibold transition-all',
                  difficulty === 'medium'
                  ? 'bg-yellow-500 text-white'
                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
                ]">Medium (4x4)</button>
                <button
                @click="changeDifficulty('hard')"
                :class="[
                  'px-4 py-2 rounded-lg font-semibold transition-all',
                  difficulty === 'hard'
                  ? 'bg-red-500 text-white'
                  : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
                ]">Hard (4x6)</button>
            </div>
          </div>

          <!-- Current Game stats -->
           <div class="bg-white rounded-lg p-6 shadow-lg">
            <p class="text-sm font-semibold text-gray-600 mb-3">Current Game</p>
            <div class="space-y-2">
              <div class="flex justify-between items-center">
                <span class="text-gray-700">Moves:</span>
                <span class="font-bold text-purple-600 text-xl">{{ moves }}</span>
              </div>
              <div class="flex justify-between items-center">
                <span class="text-gray-700">Time:</span>
                <span class="font-bold text-purple-600 text-xl">{{ formattedTime }}</span>
              </div>
            </div>
           </div>

           <!-- Best score -->
            <div v-if="getBestScore" class="bg-gradient-to-br from-yellow-400 to-orange-500 rounded-lg p-6 shadow-lg text-white">
              <p class="text-sm font-semibold mb-2">🏆 Your Best 🏆</p>
              <div class="space-y-1">
                <p class="text-lg font-bold">{{ getBestScore.moves }} moves</p>
                <p class="text-lg font-bold">{{ Math.floor(getBestScore.time / 60) }}:{{ (getBestScore.time % 60).toString().padStart(2, '0') }}</p>
                <p class="text-xl opacity-90 mt-2">{{ getBestScore.date }} at {{ getBestScore.timeOfDay }}</p>
              </div>
            </div>

            <!-- Buttons -->
             <button
              @click="newGame"
              class="bg-black text-violet-200 font-bold py-3 px-6 rounded-lg shadow-lg hover:bg-gray-100 hover:scale-105 transition-all duration-200">
            New Game</button>
            <button
              @click="toggleLeaderboard"
              class="bg-purple-600 text-white font-bold py-3 px-6 rounded-lg shadow-lg hover:bg-purple-700 transition-all">
            Leaderboard</button>
      </div>

      <!-- Center - Gameboard -->
       <div class="flex-1 flex flex-col items-center justify-center">
        <!-- win msg -->
         <div v-if="isGameWon" class="bg-green-500 text-white px-8 py-4 rounded-lg mb-6 shadow-lg animate-bounce">
          <p class="text-3xl font-bold text-center">🎊 You Won! 🎊</p>
          <p class="text-xl text-center mt-2">{{ moves }} moves in {{ formattedTime }}</p>
         </div>

         <!-- confetti animation -->
          <Confetti v-if="isGameWon" />

          <!-- Grid of cards -->
           <div
            class="grid gap-4"
            :style="{
              display: 'grid',
              gridTemplateColumns: `repeat(${gridCols}, 1fr)`,
              gap: '1rem'
            }">
            <Card
              v-for="card in cards"
              :key="card.id"
              :id="card.id"
              :emoji="card.emoji"
              :isFlipped="card.isFlipped"
              :isShaking="card.isShaking"
              @flip="handleFlip"
              />
            </div>
       </div>
  </div>

  <!-- leaderboard Modal -->
   <div
    v-if="showLeaderboard"
    class="fixed inset-0 bg-black bg-opacity-50 flex items-center jusify-center p-4 z-50"
    @click="toggleLeaderboard">
    <div
      class="bg-white rounded-lg p-8 max-w-3xl w-full max-h-[85vh] overflow-y-auto"
      @click.stop>
      <div class="flex justify-between items-center mb-6">
        <h2 class="text-4xl font-bold text-purple-600">🏆</h2>
        <button
          @click="toggleLeaderboard"
          class="text-gray-500 hover:text-gray-700 text-3xl font-bold">
          ×
        </button>
      </div>

      <!-- easy -->
       <div class="mb-8">
        <h3 class="text-2xl font-bold text-green-600 mb-3">Easy (3x4)</h3>
        <div v-if="leaderboard.filter(e => e.difficulty === 'easy').length > 0" class="space-y-2">
          <div
            v-for="(entry, index) in leaderboard.filter(e => e.difficulty === 'easy')"
            :key="index"
            class="bg-gray-100 rounded-lg p-4 flex justify-between items-center hover:bg-gray-200 transition-colors">
            <div>
              <span class="font-bold text-gray-800 text-lg">{{ index + 1 }}.</span>
              <span class="ml-3 text-gray-700">{{ entry.date }}</span>
              <span class="ml-2 text-gray-500 text-sm">{{ entry.timeOfDay }}</span>
            </div>
            <div class="text-right">
              <span class="font-semibold text-purple-600">{{ entry.moves }} moves</span>
              <span class="mx-2 text-gray-400">|</span>
              <span class="font-semibold text-blue-600">{{ Math.floor(entry.time /60) }}:{{ (entry.time % 60).toString().padStart(2, '0') }}</span>
            </div>
          </div>
        </div>
        <p v-else class="text-gray-500 italic text-center py-4">No records yet.</p>
       </div>

       <!-- medium -->
       <div class="mb-8">
        <h3 class="text-2xl font-bold text-yellow-600 mb-3">Medium (4x4)</h3>
        <div v-if="leaderboard.filter(e => e.difficulty === 'medium').length > 0" class="space-y-2">
          <div
            v-for="(entry, index) in leaderboard.filter(e => e.difficulty === 'medium')"
            :key="index"
            class="bg-gray-100 rounded-lg p-4 flex justify-between items-center hover:bg-gray-200 transition-colors">
            <div>
              <span class="font-bold text-gray-800 text-lg">{{ index + 1 }}.</span>
              <span class="ml-3 text-gray-700">{{ entry.date }}</span>
              <span class="ml-2 text-gray-500 text-sm">{{ entry.timeOfDay }}</span>
            </div>
            <div class="text-right">
              <span class="font-semibold text-purple-600">{{ entry.moves }} moves</span>
              <span class="mx-2 text-gray-400">|</span>
              <span class="font-semibold text-blue-600">{{ Math.floor(entry.time /60) }}:{{ (entry.time % 60).toString().padStart(2, '0') }}</span>
            </div>
          </div>
        </div>
        <p v-else class="text-gray-500 italic text-center py-4">No records yet.</p>
       </div>

       <!-- hard -->
       <div class="mb-8">
        <h3 class="text-2xl font-bold text-red-600 mb-3">Hard (4x6)</h3>
        <div v-if="leaderboard.filter(e => e.difficulty === 'hard').length > 0" class="space-y-2">
          <div
            v-for="(entry, index) in leaderboard.filter(e => e.difficulty === 'hard')"
            :key="index"
            class="bg-gray-100 rounded-lg p-4 flex justify-between items-center hover:bg-gray-200 transition-colors">
            <div>
              <span class="font-bold text-gray-800 text-lg">{{ index + 1 }}.</span>
              <span class="ml-3 text-gray-700">{{ entry.date }}</span>
              <span class="ml-2 text-gray-500 text-sm">{{ entry.timeOfDay }}</span>
            </div>
            <div class="text-right">
              <span class="font-semibold text-purple-600">{{ entry.moves }} moves</span>
              <span class="mx-2 text-gray-400">|</span>
              <span class="font-semibold text-blue-600">{{ Math.floor(entry.time /60) }}:{{ (entry.time % 60).toString().padStart(2, '0') }}</span>
            </div>
          </div>
        </div>
        <p v-else class="text-gray-500 italic text-center py-4">No records yet.</p>
       </div>
    </div>
  </div>
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
//leaderboard entry type
interface LeaderboardEntry {
  difficulty: Difficulty
  moves: number
  time: number
  date: string
  timeOfDay: string
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
    const leaderboard = ref<LeaderboardEntry[]>([])
    const showLeaderboard = ref(false)

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
      const now = new Date()

      addToLeaderboard({
        difficulty: difficulty.value,
        moves: moves.value,
        time: timer.value,
        date: now.toLocaleDateString(),
        timeOfDay: now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
      })
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

//load leaderboard from local storage
const loadLeaderBoard = (): void => {
  const saved = localStorage.getItem('memoryGameLeaderboard')
  if (saved) {
    leaderboard.value = JSON.parse(saved)
  }
}

//save leaderboard to local storage
const saveLeaderboard = (): void => {
  localStorage.setItem('memoryGameLeaderboard', JSON.stringify(leaderboard.value))
}

//add entry to leaderboard
const addToLeaderboard = (entry: LeaderboardEntry): void => {
  leaderboard.value.push(entry)
  //SORT BY TIME, THEN MOves
  leaderboard.value.sort((a, b) => {
    if (a.difficulty !== b.difficulty) return 0
    if (a.time !== b.time) return a.time - b.time
    return a.moves - b.moves
  })
  //keep only top 5 for each difficulty 
  const byDifficulty: Record<Difficulty, LeaderboardEntry[]> = {
    easy: [],
    medium: [],
    hard: []
  }
  leaderboard.value.forEach(entry => {
    if (byDifficulty[entry.difficulty].length < 5) {
      byDifficulty[entry.difficulty].push(entry)
    }
  })
  leaderboard.value = [...byDifficulty.easy, ...byDifficulty.medium, ...byDifficulty.hard]
  saveLeaderboard()
}

//get best score for current difficulty
const getBestScore = computed(() => {
  const scores = leaderboard.value.filter(entry => entry.difficulty === difficulty.value)
  return scores.length > 0 ? scores[0] : null
})

//toggle leaderboard visibility
const toggleLeaderboard = (): void => {
  showLeaderboard.value = !showLeaderboard.value
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

//load board when component mounts
loadLeaderBoard()

//initialize on mount
InitializeGame()
</script>