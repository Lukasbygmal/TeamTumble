<template>
  <button :class="['standard-button', color]" :style="{ fontSize: fontSize }" @click="handleClick">
    {{ label }}
  </button>
</template>

<script setup lang="ts">

defineProps<{
  label: string;
  color?: 'primary' | 'secondary' | 'danger';
  fontSize?: string;
}>();


const emit = defineEmits(['click']);
const handleClick = () => {
  emit('click');
};
</script>

<style scoped>
.standard-button {
  font-family: 'Lilita One', cursive;
  cursor: pointer;
  border: none;
  border-radius: 5px;
  padding: 3px 8px;
}

.standard-button:hover {
  filter: brightness(0.85);
}

.primary {
  background-color: #4caf50;
  color: #373434
}

.secondary {
  background-color: #0AC6C6;
  color: #373434;
}

.danger {
  background-color: #f44336;
  color: #373434;
}
</style>