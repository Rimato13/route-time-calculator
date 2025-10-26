<script setup>
defineProps({
  modelValue: Boolean,
  label: {
    type: String,
    default: '',
  },
})

defineEmits(['update:modelValue'])
</script>

<template>
  <label class="checkbox">
    <input type="checkbox" :checked="modelValue" @click.prevent />
    <span class="checkmark"></span>
    <span class="label-text">{{ label }}</span>
  </label>
</template>

<style scoped>
.checkbox {
  display: flex;
  align-items: center;
  flex-direction: row-reverse; /* text left, box right */
  gap: 0.5rem;
  cursor: pointer;
  user-select: none;
  position: relative;
}

/* Hide native checkbox */
.checkbox input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
}

/* Custom box */
.checkmark {
  width: 36px;
  height: 36px;
  border-radius: 8px;
  border: 2px solid #aaa;
  background: #fff;
  transition: all 0.25s ease;
  position: relative;
  flex-shrink: 0;
}

/* Checked state */
.checkbox input:checked + .checkmark {
  background: #1e90ff;
  border-color: #1e90ff;
  box-shadow: 0 0 6px rgba(30, 144, 255, 0.4);
}

/* Centered checkmark tick */
.checkbox input:checked + .checkmark::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 9px;
  height: 16px;
  border: solid #fff;
  border-width: 0 3px 3px 0;
  transform: translate(-50%, -50%) rotate(45deg);
  box-sizing: border-box;
}

/* Label styling */
.label-text {
  text-align: left;
  line-height: 1.2;
}
</style>
