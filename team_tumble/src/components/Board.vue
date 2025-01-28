<template>
  <div class="plinko-board">
    <canvas ref="plinkoCanvas"></canvas>
    <div v-if="resultsShown" class="results-modal">
      <div class="modal-content">
        <h2>Results</h2>
        <div class="team-results">
          <div class="team" v-for="(balls, teamId) in capturedBalls" :key="teamId" v-if="balls.length > 0">
            <h3>Team {{ teamId + 1 }}</h3>
            <ul>
              <li v-for="ball in balls" :key="ball.name">
                {{ ball.name }}
              </li>
            </ul>
          </div>
        </div>
        <button @click="closeResults">Close</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch, defineExpose } from 'vue';
import Matter, { Engine, Render, World, Bodies, Runner, Events } from 'matter-js';

const props = defineProps<{ rows: number; teams: number; balls: { id: number; name: string; color: string }[] }>();
const ballQueue = ref([...props.balls]);
const activeBalls = ref(0);
const teamCounts = ref(new Map<number, number>());
const capturedBalls = ref(Array.from({ length: 8 }, () => []));
const maxPerTeam = ref(0);
const resultsShown = ref(false);
const plinkoCanvas = ref<HTMLCanvasElement | null>(null);
let engine: Matter.Engine;
let render: Matter.Render;
let runner: Matter.Runner;

const colors = ['#ffaaaa', '#aaffaa', '#aaaaff', '#ffffaa', '#ffaaff', '#aaffff', '#ffaa00', '#aa00ff'];

const emit = defineEmits<{
  (event: 'game-ended'): void;
}>();


const setupPlinko = () => {
  const canvas = plinkoCanvas.value;
  if (!canvas) return;

  if (render) Render.stop(render);
  if (runner) Runner.stop(runner);
  if (engine) Engine.clear(engine);

  canvas.width = 800;
  canvas.height = 725;

  engine = Engine.create();
  engine.world.gravity.y = 1;

  render = Render.create({
    canvas,
    engine,
    options: {
      width: 800,
      height: 725,
      wireframes: false,
      background: '#f0f0f0'
    }
  });

  runner = Runner.create();

  const ground = Bodies.rectangle(400, 720, 800, 10, { isStatic: true, label: 'ground' });
  const leftWall = Bodies.rectangle(5, 362.5, 10, 725, { isStatic: true });
  const rightWall = Bodies.rectangle(795, 362.5, 10, 725, { isStatic: true });

  const pegs: Matter.Body[] = [];
  const rowHeights = 575;
  const pegRadius = 5;
  const boardWidth = 800;
  const pegSpacingY = rowHeights / props.rows;
  const pegSpacingX = pegSpacingY / 0.866;

  for (let row = 0; row < props.rows; row++) {
    const totalPegsInRow = row + 3;
    const rowWidth = (totalPegsInRow - 1) * pegSpacingX;
    const startX = (boardWidth - rowWidth) / 2;
    for (let col = 0; col < totalPegsInRow; col++) {
      const x = startX + col * pegSpacingX;
      const y = 100 + row * pegSpacingY;
      pegs.push(
        Bodies.circle(x, y, pegRadius, { isStatic: true, render: { fillStyle: '#000' } })
      );
    }
  }
  var num_slots = null;
  var num_slots_side = null;
  var num_empty_slots = 2;
  const slots = [];
  if (props.teams > 4) {
    num_slots = 2 * props.teams + num_empty_slots;
    num_slots_side = props.teams;
  }
  else {
    num_slots = 4 * props.teams + num_empty_slots;
    num_slots_side = 2 * props.teams;
  }

  const slotDividers = [];
  const slotWidth = (boardWidth - 20) / num_slots;

  var slot_index = 0
  for (let i = 0; i < num_slots; i++) {
    if (i < num_slots_side) {
      slots.push(
        Bodies.rectangle(
          10 + (slotWidth / 2) + i * slotWidth,
          710,
          slotWidth,
          10,
          {
            isStatic: true,
            label: `slot-${slot_index % props.teams}`,
            render: { fillStyle: colors[slot_index % props.teams] }
          }
        )
      );
      slot_index = slot_index + 1;
    }
    if (i > num_slots_side + 1) {
      slots.push(
        Bodies.rectangle(
          10 + (slotWidth / 2) + i * slotWidth,
          710,
          slotWidth,
          10,
          {
            isStatic: true,
            label: `slot-${slot_index % props.teams}`,
            render: { fillStyle: colors[slot_index % props.teams] }
          }
        )
      );
      slot_index = slot_index + 1;
    }
    if (i != 0 && i != num_slots_side + 1) {
      slotDividers.push(
        Bodies.rectangle(
          10 + i * slotWidth,
          700,
          10,
          40,
          {
            isStatic: true,
            label: 'divider',
            friction: 0.0,
            frictionStatic: 0.0,
            render: { fillStyle: '#000' }
          }
        )
      );
    }


  }

  World.add(engine.world, [ground, leftWall, rightWall, ...pegs, ...slots, ...slotDividers]);

  Events.on(engine, 'collisionStart', (event) => {
    event.pairs.forEach((pair) => {
      const { bodyA, bodyB } = pair;

      if (bodyA.label.startsWith('slot-') && bodyB.label === 'ball') {
        captureBall(bodyB, bodyA.label);
      }
      if (bodyB.label.startsWith('slot-') && bodyA.label === 'ball') {
        captureBall(bodyA, bodyB.label);
      }
      if (bodyA.label === 'ground' && bodyB.label === 'ball') {
        resetBall(bodyB);
      }
      if (bodyB.label === 'ground' && bodyA.label === 'ball') {
        resetBall(bodyA);
      }
    });
  });

  Runner.run(runner, engine);
  Render.run(render);
};

const startGame = async (balls: { id: number; name: string }[]) => {
  if (!engine) return;

  resultsShown.value = false;
  activeBalls.value = props.balls.length;
  
  ballQueue.value = [...props.balls];

  capturedBalls.value.forEach((team) => team.length = 0);
  teamCounts.value = new Map<number, number>();

  //This is buggy, when balls > teams and teams > 2 and not evenly divided 
  maxPerTeam.value = Math.ceil(balls.length / props.teams);

  for (const ball of balls) {
    spawnBall();
    await new Promise((resolve) => setTimeout(resolve, 500));
  }
};

//can duplicate ball, have to look into it
const spawnBall = () => {
  const centerX = 400;
  const slightOffset = (Math.random() - 0.2) * 20;
  const randomX = centerX + slightOffset;
  const ballData = ballQueue.value.shift();
  if (!ballData) return;

  const physicsBall = Bodies.circle(randomX, 50, 10, {
    restitution: 0.3,
    friction: 0.005,
    label: 'ball',
    render: { fillStyle: ballData.color }
  });

  physicsBall.id = ballData.id;
  World.add(engine.world, physicsBall);
};

const resetBall = (ball: Matter.Body) => {
  const ballId = ball.id;
  World.remove(engine.world, ball);

  const originalBallData = props.balls.find((b) => b.id === ballId);
  if (originalBallData) {
    ballQueue.value.push(originalBallData);
  }

  spawnBall()
};

const captureBall = (ball: Matter.Body, slotLabel: string) => {
  const teamId = parseInt(slotLabel.split('-')[1], 10);


  const teamCount = teamCounts.value.get(teamId) ?? 0;
  const ballData = props.balls.find((b) => b.id === ball.id);

  if (teamCount < maxPerTeam.value) {
    teamCounts.value.set(teamId, teamCount + 1);

    capturedBalls.value[teamId].push({
      name: ballData?.name || `${ball.id}`,
      color: ballData?.color || ball.render.fillStyle,
      ball,
    });
  }
  else {
    resetBall(ball);
    return;
  }

  World.remove(engine.world, ball);
  activeBalls.value--;
  checkResults();
};

const checkResults = () => {
  if (activeBalls.value === 0 && ballQueue.value.length === 0 && !resultsShown.value) {
    resultsShown.value = true;
    showResultsPopup();
    emit('game-ended');
  }
};

const showResultsPopup = () => {
  resultsShown.value = true;
};

const closeResults = () => {
  resultsShown.value = false;
};

watch([() => props.rows, () => props.teams], () => {
  setupPlinko();
});

defineExpose({
  setupPlinko,
  startGame,
});

onMounted(setupPlinko);
onBeforeUnmount(() => {
  if (render) Render.stop(render);
  if (runner) Runner.stop(runner);
  if (engine) Engine.clear(engine);
});
</script>

<style scoped>
.plinko-board {
  width: 800px;
  height: 725px;
  border: 1px solid #ddd;
}

.results-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

button {
  margin-top: 10px;
  padding: 10px 20px;
  font-size: 16px;
  color: white;
  background-color: #007bff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}
</style>