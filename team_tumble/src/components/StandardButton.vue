<template>
    <button
      :class="['standard-button', color]"
      :style="{ fontSize: fontSize }"
      @click="handleClick"
    >
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
  

  .primary {
    background-color: #4caf50;
    color: white;
  }
  
  .secondary {
    background-color: #008cba;
    color: white;
  }
  
  .danger {
    background-color: #f44336;
    color: white;
  }
  </style>
  