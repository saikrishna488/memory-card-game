<template>
  <div class="min-h-screen bg-gradient-to-br from-purple-500 to-pink-500 flex flex-col items-center justify-center p-4">
      <h1 class="text-5xl font-bold mb-8">Memory Card Game</h1>
      <p class="text-xl">Match pairs of cards to win!</p>

      <!-- Grid of cards -->
       <div class="gri grid-cols-4 gap-4">
        <Card
           v-for="card in cards"
           :key="card.id"
           :id="card.id"
           :emoji="card.emoji"
           :isFlipped="card.isFlipped"
           @flip="handleFlip"
           />
       </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Card from './components/Card.vue';

//Typescript interface for my card data
interface CardType {
  id: number
  emoji: string
  isFlipped: boolean
  isMatched: boolean
}

//create test cards (make dynamic later)
const cards = ref<CardType[]>([
  {id: 1, emoji: '☀️', isFlipped: false, isMatched: false},
  {id: 2, emoji: '🌙', isFlipped: false, isMatched: false},
  {id: 3, emoji: '⭐', isFlipped: false, isMatched: false},
  {id: 4, emoji: '🍀', isFlipped: false, isMatched: false},
  {id: 5, emoji: '💎', isFlipped: false, isMatched: false},
  {id: 6, emoji: '🍒', isFlipped: false, isMatched: false},
  {id: 7, emoji: '🍄‍🟫', isFlipped: false, isMatched: false},
  {id: 8, emoji: '💥', isFlipped: false, isMatched: false}
])

//Handle card flip
const handleFlip = (id: number): void => {
  const card = cards.value.find((c: CardType) => c.id === id)
  if (card)  {
    card.isFlipped = !card.isFlipped
  }
}

</script>