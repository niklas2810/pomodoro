<script setup lang="ts">
import { computed, ref, watch, watchEffect } from "vue";

//defineProps<{ msg: string }>()

const buildTime = (time: number): string => {
  const minutes = Math.floor(time / 60)
    .toString()
    .padStart(2, "0");
  const seconds = (time % 60).toString().padStart(2, "0");
  return minutes + ":" + seconds;
};

let interval = 0;

const audio = new Audio("tic-tac.mp3");

const pauseSeconds = 5;
const focusSeconds = 10;

const playEmoji = "▶️";
const pauseEmoji = "⏸️";
const stopEmoji = "⏹️";

const playing = ref(false);
const isFocus = ref(true);

const internalTime = ref(1);
const displayTime = ref("");
const sessionNumber = ref(0);

const playSound = () => {
  audio.play();
};

const reset = () => {
  isFocus.value = true;
  playing.value = false;
  internalTime.value = focusSeconds;
  sessionNumber.value = 1;
};

const setDocumentTitle = () => {
  document.title = `${
    displayTime.value
  } [${contextShort.value.toUpperCase()}] | Pomodoro Tracker`;
};

const toggleTimer = () => {
  playing.value = !playing.value;
  setDocumentTitle();
};

const onTimer = () => {
  if (internalTime.value > 0) {
    --internalTime.value;
    return;
  }

  const willPause = isFocus.value;

  internalTime.value = willPause ? pauseSeconds : focusSeconds;
  isFocus.value = !isFocus.value;

  if (isFocus.value) ++sessionNumber.value;

  playSound();
};

const startTimer = () => {
  if (interval != 0) return;

  console.log("Starting");
  interval = setInterval(onTimer, 1000);
};

const stopTimer = () => {
  if (interval == 0) return;

  console.log("Stopping");
  clearInterval(interval);
  interval = 0;
};

const contextEmoji = computed(() => {
  return playing.value ? pauseEmoji : playEmoji;
});

const contextShort = computed(() => {
  if (!playing.value) return "Pause";

  return isFocus.value ? "Focus" : "Rest";
});

const contextText = computed(() => {
  if (!playing.value) return "Paused";

  return isFocus.value ? "It's time to focus!" : "Take a rest!";
});

const progressPercent = computed(() => {
  const current = internalTime.value;
  const goal = isFocus.value ? focusSeconds : pauseSeconds;

  return 100 * (current / goal);
});

const timeFormatter = () => {
  return displayTime.value;
};

watch(playing, (newValue, oldValue) => {
  if (newValue === oldValue) return;

  if (newValue) startTimer();
  else stopTimer();
});

watch(internalTime, (newValue, oldValue) => {
  const display = buildTime(newValue);
  displayTime.value = display;
  setDocumentTitle();
});

reset();
window.addEventListener("beforeunload", (event) => {
  if (playing.value) event.preventDefault();
});
</script>

<template>
  <h3>
    Session {{ sessionNumber }}
    <span class="context-text">- {{ contextText }}</span>
  </h3>
  <h3></h3>
  <ve-progress
    :progress="progressPercent"
    :legendFormatter="timeFormatter"
    legendClass="legend-class"
    color="#42B883"
    emptyThickness="0"
  >
  </ve-progress>
  <h5>Controls</h5>
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
