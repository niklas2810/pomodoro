<script setup lang="ts">
import { computed, ref, watch, watchEffect } from 'vue'

//defineProps<{ msg: string }>()

const buildTime = (time: number): string => {
    const minutes = Math.floor(time / 60).toString().padStart(2, '0');
    const seconds = (time % 60).toString().padStart(2, '0');
    return minutes + ':' + seconds;
};

let interval = 0;

const audio = new Audio('tic-tac.mp3');

const pauseSeconds = 5;
const focusSeconds = 5;

const playEmoji = '▶️';
const pauseEmoji = '⏸️';
const stopEmoji = '⏹️';


const playing = ref(false);
const isFocus = ref(true);

const internal_time = ref(0);
const display_time = ref('');

const playSound = () => {
    audio.play(); 
}

const reset = () => {
    isFocus.value = true;
    playing.value = false;
    internal_time.value = focusSeconds;
}

const toggleTimer = () => {
    playing.value = !playing.value;
};

const onTimer = () => {
    if (internal_time.value > 0) {
        --internal_time.value;
        return;
    }

    const willPause = isFocus.value;
    
    internal_time.value = willPause ? pauseSeconds : focusSeconds;
    isFocus.value = !isFocus.value;
    playSound();
};

const startTimer = () => {
    if (interval != 0)
        return;

    console.log('Starting');
    interval = setInterval(onTimer, 1000);
};

const stopTimer = () => {
    if (interval == 0)
        return;

    console.log('Stopping');
    clearInterval(interval);
    interval = 0;
};

const contextEmoji = computed(() => {
    return playing.value ? pauseEmoji : playEmoji;
});

const contextText = computed(() => {
    if(!playing.value)
        return 'Paused';
    
    return isFocus.value ? 'It\'s time to focus!' : 'Take a rest!';
});

watch(playing, (newValue, oldValue) => {
    if (newValue === oldValue)
        return;

    if (newValue)
        startTimer();
    else
        stopTimer();
});

watch(internal_time, (newValue, oldValue) => {
    const display = buildTime(newValue);
    display_time.value = display;
    document.title = display + ' | Pomodoro Tracker';
});

reset();

</script>

<template>
    <h3 class="context-text">{{ contextText }}</h3>
    <h1>{{ display_time }}</h1>
    <button @click="toggleTimer">{{ contextEmoji }}</button>
    <button @click="reset">{{ stopEmoji }}</button>
</template>

<style scoped>
button {
    margin-inline: 10px;
}

.context-text {
    color: #888;
}
</style>
