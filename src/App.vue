<script setup lang="ts">
import { ref } from 'vue';
import modeDescription from './assets/mode-description.json';

// d=debounce, t=throttle
const onMode = ref<'d' | 't' | null>(null);
const timer = ref<number | null>(null);
const sideMap = ref<{ [key: number]: string }>({
  1: 'Side 1',
  2: 'Side 2',
  3: 'Side 3',
  4: 'Side 4',
  5: 'Side 5',
  6: 'Side 6',
});

// for setting data-side value without modifying sideMap value
// as that will cause re-render, and no scroll transition effect will happen.
const figureRefs = ref<HTMLElement>([]);

// custom directive for mouse wheel scrolling
const vWheel = {
  mounted: () => window.addEventListener('wheel', handleScroll),
};

// keys are the sequence of the sides
// their value is correspond to the sideMap's keys
const sideSequence = {
  1: 1,
  2: 2,
  3: 3,
  4: 4,
  5: 5,
  6: 6,
};
const sequenceKey = Object.keys(sideSequence);

function updateSideSequence() {
  // console.log('scroll');
  sequenceKey.forEach((key) => {
    if (sideSequence[key] - 1 > 0) {
      sideSequence[key] -= 1;
    } else {
      sideSequence[key] = sequenceKey.length;
    }
    const newSeq = sideSequence[key];
    // console.log(newSeq);
    figureRefs.value[key - 1].dataset.side = newSeq;
  });
}

function handleScroll() {
  switch (onMode.value) {
    case 'd':
      clearTimeout(timer.value);
      timer.value = setTimeout(() => {
        console.log('leave debounce');
        updateSideSequence();
        timer.value = null;
      }, 1500);
      break;
    case 'di':
      if (timer.value === null) {
        updateSideSequence();
      }
      clearTimeout(timer.value);
      timer.value = setTimeout(() => {
        console.log('leave debounce with leading flag');
        timer.value = null;
      }, 1500);
      break;
    case 't':
      if (timer.value) {
        break;
      }
      updateSideSequence();
      timer.value = setTimeout(() => {
        console.log('leave throttle');
        clearTimeout(timer.value);
        timer.value = null;
      }, 2000);
      break;
    default:
      updateSideSequence();
  }
}

function handleClick(mode) {
  onMode.value = onMode.value === mode ? null : mode;
}
</script>

<template>
  <div class="container">
    <h1>Debounce & Throttle</h1>
    <div class="mode-container">
      <p>Click the buttons to selecto or deselect</p>
      <h2>Mode Selector</h2>
      <div class="mode-btns">
        <button
          :class="{
            'current-mode': onMode === 'd',
            twinkle: onMode === 'd' && timer,
          }"
          @click="handleClick('d')"
        >
          Debounce
        </button>
        <button
          :class="{
            'current-mode': onMode === 'di',
            twinkle: onMode === 'di' && timer,
          }"
          @click="handleClick('di')"
        >
          Debounce
          <span>immediate</span>
        </button>
        <button
          :class="{
            'current-mode': onMode === 't',
            twinkle: onMode === 't' && timer,
          }"
          @click="handleClick('t')"
        >
          Throttle
        </button>
      </div>
    </div>
    <div class="carousel-wrapper" v-wheel>
      <span>Try scrolling your mouse wheel👇</span>
      <div class="carousel" data-state="1">
        <figure
          v-for="seq in Object.keys(sideMap)"
          :data-side="seq"
          ref="figureRefs"
        >
          {{ sideMap[seq] }}
        </figure>
      </div>
    </div>
    <div class="description-container">
      <h2>Mode Description</h2>
      <p>{{ modeDescription[onMode] }}</p>
    </div>
  </div>
</template>

<style scoped>
.container {
  --container-width: 16rem;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  width: var(--container-width);
  height: 100%;
}

.mode-container {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  width: 100%;
}

.mode-btns {
  display: flex;
  justify-content: center;
  gap: 0.25rem;
}

.current-mode {
  background-color: green;
}

/* the carousel's container, doesn't affect the trasition effects */
.carousel-wrapper {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  align-items: center;
  height: 10rem;
  width: var(--container-width);
  border: 1px solid white;
}

.carousel-wrapper > span {
  margin-top: 0.25rem;
}

.carousel {
  --carousel-side-1: rotateX(0deg);
  --carousel-side-2: rotateX(60deg) translateY(-10px) translateZ(30px);
  --carousel-side-3: rotateX(120deg) translateY(-10px) translateZ(40px);
  --carousel-side-4: rotateX(180deg);
  --carousel-side-5: rotateX(240deg) translateY(10px) translateZ(40px);
  --carousel-side-6: rotateX(300deg) translateY(10px) translateZ(30px);
  position: relative;
  perspective: 1000px;
  transform: translateY(50%);
  width: 100%;
  height: 100%;
}

.carousel figure {
  /* to transform regularly, the sides should start from the a same position */
  position: absolute;
  width: 100%;
  margin: 0;
  backface-visibility: hidden;
  font-size: 1.2rem;
  transition: all 1s;
  opacity: 0.4;
}

/* 
 set the position corresponds with custom properties.
 only the item with data-side 1 has full opacity
*/
.carousel > figure[data-side='1'] {
  -webkit-transform: var(--carousel-side-1);
  opacity: 1;
}
.carousel > figure[data-side='2'] {
  -webkit-transform: var(--carousel-side-2);
}
.carousel > figure[data-side='3'] {
  -webkit-transform: var(--carousel-side-3);
}
.carousel > figure[data-side='4'] {
  -webkit-transform: var(--carousel-side-4);
}
.carousel > figure[data-side='5'] {
  -webkit-transform: var(--carousel-side-5);
}
.carousel > figure[data-side='6'] {
  -webkit-transform: var(--carousel-side-6);
}

.description-container {
  width: fit-content;
}

.description-container > h2 {
  margin: auto;
  width: max-content;
}

.description-container > p {
  padding-inline: 0.75rem;
  text-align: left;
  height: 8rem;
}

.twinkle {
  animation-name: twinkle;
  animation-iteration-count: infinite;
  animation-duration: 1s;
}

@keyframes twinkle {
  0% {
    background-color: Olive;
  }
  50% {
    background-color: Yellow;
    opacity: 0.5;
  }
  100% {
    background-color: Olive;
  }
}
</style>
