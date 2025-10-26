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
  <label class="combo">
    <span>{{ label }}</span>
    <input :list="id" v-model="model" class="combo-input" />
    <datalist :id="id">
      <option v-for="(option, i) in options" :key="i" :value="option.value">
        {{ option.label }}
      </option>
    </datalist>
  </label>
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
  border-bottom: 2px solid rgba(0, 0, 0, 0.3);
  outline: none;
  background: transparent;
  font-size: 1rem;
  padding: 0.25rem 0;
  transition: border-color 0.2s;
}

.combo-input:focus {
  border-bottom-color: #1e90ff;
}
</style>
