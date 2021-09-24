<template>
  <Vue2048 ref="vue2048" @no-change="onNoChange" @point="onAddPoint" />
  <p>Score: {{ totalPoint }}, Best Score: {{ bestPoint }}</p>
  <h1>Vue Slide Puzzle</h1>

  <!-- Control -->
  <div class="panel">
    <div class="reset-btn" @click="reset2048" touchstart="">
      <span>Reset</span>
    </div>
  </div>

  <!-- Social -->
  <a
    href="https://github.com/johnnywang1994/vue-2048/tree/master/src/components/Vue2048.vue"
    target="_blank"
  >
    Source Code
  </a>
</template>

<script setup>
import { ref, watch } from 'vue'
import Vue2048 from './components/Vue2048.vue'

const vue2048 = ref(null);
const totalPoint = ref(0);
const bestPoint = ref(localStorage['best-score'] || 0);

watch(totalPoint, (newPoint) => {
  if (newPoint > bestPoint.value) {
    bestPoint.value = newPoint;
    localStorage['best-score'] = String(newPoint);
  }
});

function onNoChange() {
  console.log('no change');
}

function onAddPoint(point) {
  totalPoint.value += point;
}

function reset2048() {
  totalPoint.value = 0;
  vue2048.value.onResetGame();
}
</script>

<style lang="scss">
@mixin activeScale {
  cursor: pointer;
  transition: transform 0.2s, filter 0.2s;
  &:active {
    transform: scale(0.95);
    filter: brightness(0.5);
  }
}

@mixin noTouchOrSelect {
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  user-select: none;
}

@mixin btn_style {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 100px;
  height: 40px;
  font-weight: 700;
  border-radius: 10px;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.reset-btn {
  color: #fff;
  border: 1px solid rgb(0, 163, 128);
  background: rgb(10, 206, 163);
  @include btn_style;
  @include activeScale;
  @include noTouchOrSelect;
}
</style>
