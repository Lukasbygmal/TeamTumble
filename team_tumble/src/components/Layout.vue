<template>
  <div class="plinko-container">
    <div class="sidebar">
      <StandardButton label="Start" color="primary" @click="startGame" />
      
      <div>
        <label for="rowSlider">Rows: {{ sliderValue }}</label>
        <input
          id="rowSlider"
          type="range"
          min="12"
          max="22"
          v-model="sliderValue"
        />
        <StandardButton label="Apply" color="primary" @click="applyRowChange" />
      </div>
      
    </div>

    
    <Board ref="plinkoBoard" :rows="rows" />

    <div class="sidebar">
      <StandardButton label="Add Ball" color="secondary" @click="addBall" />
      <div class="ball-list">
        <div
          v-for="(ball, index) in balls"
          :key="ball.id"
          class="ball-item"
        >
          <input
            v-model="ball.name"
            :placeholder="`Ball ${index + 1}`"
          />
          <StandardButton label="-" color="danger" @click="removeBall(index)" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import StandardButton from '@/components/StandardButton.vue';
import Board from '@/components/Board.vue';

const sliderValue = ref(18);
const rows = ref(18);
const balls = ref<{ id: number; name: string }[]>([]);
let ballId = 1;

const plinkoBoard = ref<InstanceType<typeof Board> | null>(null);

const applyRowChange = () => {
  rows.value = sliderValue.value;
  plinkoBoard.value?.setupPlinko();
};

const startGame = () => {
  plinkoBoard.value?.startGame(balls.value);
};

const addBall = () => {
  balls.value.push({
    id: ballId++,
    name: `Ball ${balls.value.length + 1}`
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
  margin: 20px auto;
}

.sidebar {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.sidebar input[type="range"] {
  margin-bottom: 10px;
}

.ball-list {
  height: 600px;
  overflow-y: auto;
  border: 1px solid #ccc;
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
</style>
