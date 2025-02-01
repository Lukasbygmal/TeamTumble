<template>
  <div class="plinko-container">
    <div class="sidebar">
  
    </div>
    <Board ref="plinkoBoard" :rows="rows" :teams="teams" :balls="balls" @game-ended="handleGameEnd" />
    <div class="sidebar">
      <StandardButton :label="isGameActive ? 'Cancel' : 'Start'" :color="isGameActive ? 'danger' : 'primary'"
        fontSize="48px" @click="isGameActive ? cancelGame() : startGame()" />
      <div class="ball-list">
        <div v-for="(ball, index) in balls" :key="ball.id" class="ball-item">
          <div class="color-indicator" :style="{ backgroundColor: ball.color }"></div>
          <input v-model="ball.name" :placeholder="`Ball ${index + 1}`" />
          <StandardButton label="-" color="danger" fontSize="16px" @click="removeBall(index)"
            :disabled="isGameActive" />
        </div>
      </div>
      <StandardButton label="Add Ball" color="secondary" fontSize="24px" @click="addBall" :disabled="isGameActive" />
      <CustomSlider id="rowSlider" label="Rows" :min="12" :max="22" :disabled="isGameActive" v-model="rowSliderValue" />
      <CustomSlider id="teamSlider" label="Teams" :min="2" :max="8" :disabled="isGameActive"
        v-model="teamSliderValue" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue';
import StandardButton from '@/components/StandardButton.vue';
import Board from '@/components/Board.vue';
import CustomSlider from './CustomSlider.vue';

const isGameActive = ref(false);
const rowSliderValue = ref(16);
const teamSliderValue = ref(2);
const rows = ref(16);
const teams = ref(2);
const balls = ref<{ id: number; name: string, color: string }[]>([]);
let ballId = 0;

const plinkoBoard = ref<InstanceType<typeof Board> | null>(null);

watch(rowSliderValue, (newValue) => {
  rows.value = Number(newValue);
  plinkoBoard.value?.setupPlinko();
});

watch(teamSliderValue, (newValue) => {
  teams.value = Number(newValue);
  plinkoBoard.value?.setupPlinko();
});

const startGame = () => {
  plinkoBoard.value?.startGame(balls.value);
  isGameActive.value = true;
};

const cancelGame = () => {
  plinkoBoard.value?.cancelGame();
  isGameActive.value = false;
};

const handleGameEnd = () => {
  isGameActive.value = false;
};

const addBall = () => {
  balls.value.push({
    id: ballId++,
    name: "" + ballId,
    color: `#${Math.floor(Math.random() * 16777215).toString(16).padStart(6, '0')}`,
  });
};

const removeBall = (index: number) => {
  balls.value.splice(index, 1);
};
</script>

<style scoped>
.plinko-container {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  gap: 20px;
}

.sidebar {
  width: 250px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}


.ball-list {
  height: 400px;
  overflow-y: auto;
  width: 250px;
  padding: 5px;
}

.ball-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin: 5px 0;
}

.ball-item input {
  flex-grow: 1;
  margin-right: 5px;
  padding: 2px;
}

.color-indicator {
  width: 24px;
  height: 24px;
  border-radius: 4px;
  margin: 2px;
}
</style>
