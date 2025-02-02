<template>
  <div class="plinko-board">
    <canvas ref="plinkoCanvas"></canvas>
    <div v-if="resultsShown" class="results-modal">
      <div class="modal-content">
        <h2>Results</h2>
        <div class="team-results">
          <div v-for="(balls, teamId) in capturedBalls" v-show="teamId < teams" :key="teamId" class="team">
            <h3>Team {{ teamId + 1 }}</h3>
            <ul>
              <li v-for="ball in balls" :key="ball.name">
                {{ ball.name }}
              </li>
            </ul>
          </div>
        </div>
        <StandardButton label="Close" fontSize="24px" color="primary" @click="closeResults" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch, defineExpose } from 'vue';
import Matter, { Engine, Render, World, Bodies, Runner, Events } from 'matter-js';
import StandardButton from '@/components/StandardButton.vue';

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
      background: '#d3d3d3'
    }
  });

  runner = Runner.create();

  const ground = Bodies.rectangle(400, 720, 800, 10, { isStatic: true, render: { fillStyle: '#373434' } });
  const leftWall = Bodies.rectangle(5, 362.5, 10, 725, { isStatic: true, render: { fillStyle: '#373434' } });
  const rightWall = Bodies.rectangle(795, 362.5, 10, 725, { isStatic: true, render: { fillStyle: '#373434' } });

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
    else if (i > num_slots_side + 1) {
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
    if (i == num_slots_side) {
      slots.push(
        Bodies.rectangle(
          10 + (slotWidth) + i * slotWidth,
          710,
          slotWidth * 2,
          10,
          {
            isStatic: true,
            label: 'ground',
            render: { fillStyle: '#000' } //this should invisible
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
            render: { fillStyle: '#373434' }
          }
        )
      );
      slotDividers.push(
        Bodies.circle(
          10 + i * slotWidth,
          700 - 20,
          5,
          {
            isStatic: true,
            label: 'divider',
            friction: 0.0,
            frictionStatic: 0.0,
            render: { fillStyle: '#373434' }
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

  //When balls > teams and teams > 2 and not evenly divided, it will cause "issues", but sorta how it will have to work 
  maxPerTeam.value = Math.ceil(balls.length / props.teams);

  for (const ball of balls) {
    spawnBall();
    await new Promise((resolve) => setTimeout(resolve, 500));
  }
};

const spawnBall = () => {
  const centerX = 400;
  const slightOffset = (Math.random() - 0.5) * 20;
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

  const isCaptured = capturedBalls.value.some(team => team.some(b => b.ball.id === ball.id));
  if (isCaptured) return;

  World.remove(engine.world, ball);

  const originalBallData = props.balls.find((b) => b.id === ballId);
  if (originalBallData) {
    ballQueue.value.push(originalBallData);
  }

  spawnBall()
};

const captureBall = (ball: Matter.Body, slotLabel: string) => {
  const teamId = parseInt(slotLabel.split('-')[1], 10);

  if (capturedBalls.value[teamId].some(b => b.ball.id === ball.id)) return;

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

const cancelGame = () => {
  if (render) Render.stop(render);
  if (runner) Runner.stop(runner);
  if (engine) Engine.clear(engine);

  ballQueue.value = [];
  activeBalls.value = 0;
  capturedBalls.value.forEach((team) => team.length = 0);
  teamCounts.value.clear();
  resultsShown.value = false;

  setupPlinko();
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
  cancelGame,
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
  background: #d3d3d3;
  padding: 10px;
  border-radius: 8px;
  text-align: center;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  font-family: 'Lilita One', cursive;
  font-size: 18px;
}

.modal-content h2 {
  margin-bottom: 0px;
}

.team-results {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
  max-width: 800px;
  margin-bottom: 10px;
}


.team {
  padding: 10px;
}


.team h3 {
  margin-bottom: 5px;
}

.team ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.team li {
  margin-bottom: 5px;
}

</style>