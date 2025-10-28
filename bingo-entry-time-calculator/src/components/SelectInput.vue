<script setup>
import { computed } from 'vue'
import { useId } from 'vue'

const props = defineProps({
  modelValue: String,
  label: String,
  options: {
    type: Array,
    default: () => [],
  },
})

const emit = defineEmits(['update:modelValue'])

const model = computed({
  get: () => props.modelValue,
  set: (val) => emit('update:modelValue', val),
})

const id = useId()
</script>

<template>
  <div class="combo">
    <label :for="id" class="combo-label">{{ label }}</label>
    <input :list="id" v-model="model" class="combo-input" />
    <datalist :id="id">
      <option v-for="(option, i) in options" :key="i" :value="option.value">
        {{ option.label }}
      </option>
    </datalist>
  </div>
</template>

<style scoped>
.combo {
  display: flex;
  flex-direction: column;
  gap: 4px;
  width: 175px;
}

.combo-input {
  border: none;
  border-bottom: 2px solid #1e90ff;
  outline: none;
  background: transparent;
  font-size: 1rem;
  padding: 0.25rem 0;
  transition: border-color 0.2s;
}

.combo {
  display: flex;
  flex-direction: column;
  gap: 4px;
  width: 200px;
}

.combo-label {
  font-weight: 600;
  color: #333;
  text-shadow: 1px 1px 0 rgba(255, 255, 255, 0.4);
  font-size: 0.95rem;
  line-height: 1.2;
  white-space: 'nowrap';
}
</style>
