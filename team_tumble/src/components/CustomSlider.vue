<template>
    <div class="slider-container">
        <label :for="id" class="slider-label">{{ label }}: {{ modelValue }}</label>
        <input :id="id" type="range" :min="min" :max="max" :disabled="disabled" :value="modelValue" @input="handleInput"
            class="slider" />
    </div>
</template>

<script setup lang="ts">
import { defineProps, defineEmits } from 'vue';

const props = defineProps<{
    id: string;
    label: string;
    min: number;
    max: number;
    disabled: boolean;
    modelValue: number;
}>();

const emit = defineEmits<{
    (e: 'update:modelValue', value: number): void
}>();

const handleInput = (event: Event) => {
    const target = event.target as HTMLInputElement;
    emit('update:modelValue', +target.value);
};
</script>

<style scoped>
.slider-container {
    width: 100%;
    max-width: 300px;
    margin: 10px auto;
}

.slider-label {
    font-family: 'Lilita One', cursive;
    display: block;
    margin-bottom: 5px;
    color: #0AC6C6;
    font-size: 1.2em;
}

.slider {
    width: 100%;
    height: 12px;
    border-radius: 5px;
    background: #d3d3d3;
    outline: none;
}

.slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: #4caf50;
    cursor: pointer;
    transition: background .2s ease-in-out;
}

.slider::-moz-range-thumb {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: #0AC6C6;
    cursor: pointer;
    transition: background .2s ease-in-out;
    border: none;
}

.slider::-webkit-slider-thumb:hover {
    background: #45a049;
}

.slider:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}

.slider:disabled::-webkit-slider-thumb {
    background: #cccccc;
    cursor: not-allowed;
}

.slider:disabled::-moz-range-thumb {
    background: #cccccc;
    cursor: not-allowed;
}
</style>