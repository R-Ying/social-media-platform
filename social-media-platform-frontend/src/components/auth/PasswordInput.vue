<template>
  <div class="form-group">
    <label :for="id || 'password'">{{ label }}</label>
    <div class="password-input-container">
      <input
        :id="id || 'password'"
        v-model="modelValue"
        :type="showPassword ? 'text' : 'password'"
        :placeholder="placeholder"
        class="form-input"
        :class="{ 'is-error': error }"
        @input="$emit('update:modelValue', modelValue)"
      />
      <button
        type="button"
        class="toggle-password-btn"
        @click="showPassword = !showPassword"
      >
        <span v-if="showPassword">隱藏</span>
        <span v-else>顯示</span>
      </button>
    </div>
    <p v-if="error" class="error-message">{{ error }}</p>
    <p v-if="hint" class="input-hint">{{ hint }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue';

interface Props {
  modelValue: string;
  label?: string;
  placeholder?: string;
  error?: string;
  hint?: string;
  id?: string;
}

interface Emits {
  (e: 'update:modelValue', value: string): void;
}

const props = withDefaults(defineProps<Props>(), {
  label: '密碼',
  placeholder: '請輸入密碼',
  error: '',
  hint: '',
  id: 'password'
});

const modelValue = ref(props.modelValue);
const showPassword = ref(false);

watch(() => props.modelValue, (newVal) => {
  modelValue.value = newVal;
});
</script>

<style lang="scss" scoped>
@use "@/assets/styles/forms.scss";

.password-input-container {
  position: relative;

  .form-input {
    padding-right: 60px; // Make room for the toggle button
  }

  .toggle-password-btn {
    position: absolute;
    right: 12px;
    top: 50%;
    transform: translateY(-50%);
    background: none;
    border: none;
    color: #666;
    cursor: pointer;
    font-size: 14px;

    &:hover {
      color: #4a90e2;
    }
  }
}

.input-hint {
  margin-top: 4px;
  font-size: 14px;
  color: #666;
}
</style>
