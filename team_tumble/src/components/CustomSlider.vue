<template>
    <div class="slider-container">
        <label :for="id" class="slider-label">{{ label }}: {{ modelValue }}</label>
        <input :id="id" type="range" :min="min" :max="max" :disabled="disabled" :value="modelValue"
            @input="$emit('update:modelValue', +$event.target.value)" class="slider" />
    </div>
</template>

<script setup lang="ts">
defineProps<{
    id: string;
    label: string;
    min: number;
    max: number;
    disabled: boolean;
    modelValue: number;
}>();

defineEmits(['update:modelValue']);
</script>

<style scoped>
.slider-container {
    margin: 20px 0;
    width: 100%;
    max-width: 300px;
    margin: 20px auto;
}

.slider-label {
    font-family: 'Lilita One', cursive;
    display: block;
    margin-bottom: 10px;
    color: #0AC6C6; 
    font-size: 1.2em;
}

.slider {
    -webkit-appearance: none;
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