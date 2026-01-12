<template>
    <div class="card relative w-24 h-24 cursor-pointer perspective" :class="{ 'shake': isShaking }" @click="handleClick">
        <div class="card-inner w-full h-full transition-transform duraion-500 transform-style-3d"
             :class="{ 'rotate-y-180': isFlipped }">
            <!-- Card Back (face down) -->
             <div class="card-face absolute w-full h-full backface-hidden bg-gradient-to-br from-slate-950 to-purple-600 rounded-lg shadow-lg flex items-center justify-center">
                <span class="text-4xl">🎴</span>
             </div>

            <!-- Card Front (face up) -->
            <div class="card-face absolute w-full h-full backface-hidden bg-indigo-200 rounded-lg shadow-lg flex items-center justify-center rotate-y-180">
                <span class="text-4xl">{{ emoji }}</span>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">

    // Define props that the component accepts
    interface Props {
        id: number;
        emoji: string;
        isFlipped: boolean;
        isShaking?: boolean;
    }

    //Accepts props with types
    const props = defineProps<Props>();

    // Define events this component can emit
    const emit = defineEmits<{
        (e: 'flip', id: number): void
    }>();

    //Handle card click
    const handleClick = () => {
        if (!props.isFlipped) {
            emit('flip', props.id);
        }
    };
</script>

<style scoped>
.perspective {
    perspective: 1000px;
}

.transform-style-3d {
    transform-style: preserve-3d;
}

.backface-hidden {
    backface-visibility: hidden;
}

.rotate-y-180 {
    transform: rotateY(180deg);
}

/* Shake animation */
.shake {
    animation: shake 0.5s;
}

@keyframes shake {
    0%, 100% { transform: translateX(0); }
    10%, 30%, 50%, 70%, 90% { transform: translateY(-5px); }
    20%, 40%, 60%, 80% { transform: translateX(5px); }
}
</style>