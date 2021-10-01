<template>
  <div class="playground">
    <div v-for="(row, ri) in cardList" :key="`card-row-${ri}`" class="card-row">
      <div v-for="(card, i) in row" :key="`card-${i}`" class="card">
        <div
          :class="['card-grid', { bounce: card.value > 0 }]"
          :key="`card-grid-${i}-${card.value}`"
          :data-value="card.value"
        >
          {{ card.value > 0 ? card.value : '' }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';

// -- Emits
// eslint-disable-next-line
const emit = defineEmits([
  'no-change',
  'point',
]);

// eslint-disable-next-line
defineExpose({
  onResetGame,
})

// -- Tools
const cardChanged = ref(false);
const [direction] = useTouchSlide('.playground');

const getNextMap = {
  1: (r, c) => () => r > 0 && cardList.value[--r][c],
  2: (r, c) => () => cardList.value[r][++c],
  3: (r, c) => () => r < 3 && cardList.value[++r][c],
  4: (r, c) => () => cardList.value[r][--c],
};

const getLoopMap = {
  1: onSlideTop,
  2: onSlideRight,
  3: onSlideDown,
  4: onSlideLeft,
};


// -- Cards
// [[], [], [], []]
const cardList = ref([]);


// -- Events
function onAddCard(cardNumber, list) {
  if (cardNumber <= 0) return;

  const indexList = list || cardList.value
    .reduce(
      (list, row, i) => list.concat(row.map((card, j) => ({
        i,
        j,
        ...card
      }))),
      [],
    )
    .filter((card) => card.value == 0);

  if (cardNumber > indexList.length) return;

  const index = Math.floor(Math.random() * indexList.length);
  const target = cardList.value[indexList[index].i][indexList[index].j];
  target.value = 2;
  indexList.splice(index, 1);

  return onAddCard(--cardNumber, indexList);
}

function onMove(direction, r, c) {
  let next;
  let card = cardList.value[r][c];
  // card exist
  if (card.value > 0) {
    const getNext = getNextMap[direction](r, c);
    next = getNext();
    // move
    while (next && next.value == 0) {
      next.value = card.value;
      card.value = 0;
      card = next;
      next = getNext();
      cardChanged.value = true;
    }
    // merge
    if (next && next.value === card.value) {
      next.value *= 2;
      card.value = 0;
      cardChanged.value = true;
      // add value
      emit('point', next.value);
    }
  }
}

function onSlideRight(direction) {
  for (let r = 0; r < 4; r++) {
    for (let c = 3; c >= 0; c--) {
      onMove(direction, r, c);
    }
  }
}

function onSlideLeft(direction) {
  for (let r = 0; r < 4; r++) {
    for (let c = 0; c < 4; c++) {
      onMove(direction, r, c);
    }
  }
}

function onSlideTop(direction) {
  for (let c = 0; c < 4; c++) {
    for (let r = 0; r < 4; r++) {
      onMove(direction, r, c);
    }
  }
}

function onSlideDown(direction) {
  for (let c = 0; c < 4; c++) {
    for (let r = 3; r >= 0; r--) {
      onMove(direction, r, c);
    }
  }
}

function onResetGame() {
  cardList.value = Array(4).fill('').map(
    () => Array(4).fill('').map(() => ({
      value: 0
    }))
  );
  onAddCard(2);
}

watch(direction, (newVal) => {
  const loopFn = getLoopMap[newVal];
  if (loopFn) {
    loopFn(newVal);
    if (cardChanged.value) {
      onAddCard(1);
      cardChanged.value = false;
    } else {
      emit('no-change');
    }
  }
})

// -- Init
onResetGame();


function useTouchSlide(selector, safeOffset = 20) {
  // 1: top, 2: right, 3: down, 4: left
  const direction = ref(0);
  
  let xDown = null;
  let yDown = null;
  
  function handleTouchStart(evt) {
    const firstTouch = (evt.touches && evt.touches[0]) || evt;
    xDown = firstTouch.clientX;
    yDown = firstTouch.clientY;
  }
  
  function handleTouchStop() {
    xDown = yDown = null;
    direction.value = 0;
  }
  
  function handleTouchMove(evt) {
    if (!xDown || !yDown) return;
    
    const currentTouch = (evt.touches && evt.touches[0]) || evt;
    const xUp = currentTouch.clientX;
    const yUp = currentTouch.clientY;
    
    const xDiff = xDown - xUp;
    const yDiff = yDown - yUp;
    
    if (Math.abs(xDiff) > Math.abs(yDiff)) {
      if (Math.abs(xDiff) < safeOffset) return;
      direction.value = xDiff > 0 ? 4 : 2;
    } else {
      if (Math.abs(yDiff) < safeOffset) return;
      direction.value = yDiff > 0 ? 1 : 3;
    }
  }
  
  function init() {
    const target = document.querySelector(selector);
    target.addEventListener('touchstart', handleTouchStart, false);
    target.addEventListener('touchmove', handleTouchMove, false);
    target.addEventListener('touchend', handleTouchStop, false);
    target.addEventListener('mousedown', handleTouchStart, false);
    target.addEventListener('mousemove', handleTouchMove, false);
    target.addEventListener('mouseup', handleTouchStop, false);
    target.addEventListener('mouseleave', handleTouchStop, false);
  }
  
  function destroy() {
    const target = document.querySelector(selector);
    target.removeEventListener('touchstart', handleTouchStart);
    target.removeEventListener('touchmove', handleTouchMove);
    target.removeEventListener('touchend', handleTouchStop);
    target.removeEventListener('mousedown', handleTouchStart);
    target.removeEventListener('mousemove', handleTouchMove);
    target.removeEventListener('mouseup', handleTouchStop);
  }
  
  onMounted(() => {
    init();
  })
  
  return [direction, destroy];
}
</script>

<style lang="scss">
* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

@keyframes bounce {
  0% {
    transform: scale(1);
  }
  70% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

@function pow($base, $exponents) {
  @if $exponents == 0 {
    @return 0;
  }
  
  $raised: 1;

  @for $i from 1 through $exponents {
    $raised: $raised * $base;
  }

  @return $raised;
}

$bgcolor-array: (
  rgba(238, 228, 218, 0.35),
  #faf8ef,
  #eee1c9,
  #f3b27a,
  #f69664,
  #f77c5f,
  #f75f3b,
  #edd073,
  #edcc62,
  #edc950,
  #edc53f,
  #edc22e, // 2048
);

.playground {
  display: flex;
  flex-wrap: wrap;
  width: 300px;
  height: 300px;
  margin: auto;
  background: #bbada0;
  user-select: none;
  cursor: grab;
  > .card-row {
    display: flex;
    flex-wrap: wrap;
    width: 100%;
    height: 25%;
    > .card {
      width: 25%;
      height: 100%;
      font-size: 30px;
      font-weight: bold;
      padding: 5px;
      > .card-grid {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100%;
        color: #776e65;
        &.bounce {
          animation: bounce 0.2s ease-in 1;
        }
        @for $i from 1 through length($bgcolor-array) {
          &[data-value='#{pow(2, $i - 1)}'] {
            background: nth($bgcolor-array, $i);
            @if pow(2, $i - 1) >= 32 {
              color: #f9f6f2;
            }
          }
        }
      }
    }
  }
}
</style>
