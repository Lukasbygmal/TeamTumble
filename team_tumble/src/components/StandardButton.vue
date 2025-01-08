<template>
    <button
      :class="['standard-button', color]"
      @click="handleClick"
    >
      {{ label }}
    </button>
  </template>
  
  <script setup lang="ts">

  defineProps<{
    label: string;
    color?: 'primary' | 'secondary' | 'danger';
  }>();
  

  const emit = defineEmits(['click']);
  const handleClick = () => {
    emit('click');
  };
  </script>
  
  <style scoped>
  .standard-button {
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
    border: none;
    border-radius: 5px;
    margin-bottom: 10px;
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
  