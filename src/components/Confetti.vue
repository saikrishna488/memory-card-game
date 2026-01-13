<template>
    <div class="confetti-container">
        <div v-for="i in 50" :key="i" class="confetti" :style="getConfettiStyle(i)">
        </div>
    </div>
</template>

<script setup lang="ts">
    const colors = ['#ff0000', '#00ff00', '#0000ff', '#ffff00', 'ff00ff', '00ffff', '#ffa500', '#ff1493']

    const getConfettiStyle = (index: number) => {
        const color = colors[index % colors.length]
        const left = Math.random() * 100
        const animationDelay = Math.random() * 3
        const animationDuration = 3 + Math.random() * 2

        return {
            backgroundColor: color,
            left: `${left}%`,
            animationDelay: `${animationDelay}s`,
            animationDuration: `${animationDuration}s`
        }
    }
</script>

<style scoped>
    .confetti-container {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        pointer-events: none;
        overflow: hidden;
        z-index: 9999;
    }

    .confetti {
        position: absolute;
        width: 10px;
        height: 10px;
        top: -10px;
        opacity: 0;
        animation: confetti-fall linear forwards;
    }

    @keyframes confetti-fall {
        0% {
            top: -10px;
            opacity: 1;
            transform: translateX(0) rotateZ(0deg);
        }
        100% {
            top: 100vh;
            opacity: 0;
            transform: translateX(100px) rotateZ(720deg);
        }
    }
</style>