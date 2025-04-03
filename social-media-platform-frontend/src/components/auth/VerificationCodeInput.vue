<template>
  <div class="form-group">
    <label :for="id">{{ label }}</label>
    <div class="verification-container">
      <input
        :id="id"
        v-model="modelValue"
        type="text"
        :placeholder="placeholder"
        class="form-input"
        :class="{ 'is-error': error }"
        @input="$emit('update:modelValue', modelValue)"
      />
      <button
        type="button"
        class="verification-btn"
        :disabled="isCountingDown"
        @click="onSendCode"
      >
        {{ countdownText }}
      </button>
    </div>
    <p v-if="error" class="error-message">{{ error }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue';

interface Props {
  modelValue: string;
  label?: string;
  placeholder?: string;
  error?: string;
  id?: string;
  countdownText: string;
  isCountingDown: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  label: '驗證碼',
  placeholder: '請輸入驗證碼',
  error: '',
  id: 'verificationCode'
});

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void;
  (e: 'send-code'): void;
}>();

const modelValue = ref(props.modelValue);

watch(() => props.modelValue, (newVal) => {
  modelValue.value = newVal;
});

// 發送驗證碼
const onSendCode = () => {
  emit('send-code');
};
</script>

<style lang="scss" scoped>
@use "@/assets/styles/forms.scss";

.verification-container {
  display: flex;
  width: 100%;

  .form-input {
    flex: 1;
    border-radius: 8px 0 0 8px;
  }

  .verification-btn {
    min-width: 120px;
    background-color: #f5f5f5;
    border: 1px solid #ddd;
    border-radius: 0 8px 8px 0;
    padding: 0 12px;
    color: #333;
    cursor: pointer;
    white-space: nowrap;
    transition: background-color 0.3s, color 0.3s;

    &:hover:not(:disabled) {
      background-color: #e9e9e9;
    }

    &:disabled {
      background-color: #f9f9f9;
      color: #999;
      cursor: not-allowed;
    }
  }
}
</style>
