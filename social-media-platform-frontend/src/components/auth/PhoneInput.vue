<template>
  <div class="form-group">
    <label for="phone">手機號碼</label>
    <div class="phone-input-container">
      <select
        v-model="modelCountryCode"
        class="country-code"
        @change="emitUpdate"
      >
        <option v-for="code in countryCodes" :key="code" :value="code">{{ code }}</option>
      </select>
      <input
        id="phone"
        v-model="modelPhone"
        type="tel"
        placeholder="請輸入手機號碼"
        class="form-input"
        :class="{ 'is-error': error }"
        @input="emitUpdate"
      />
    </div>
    <p v-if="error" class="error-message">{{ error }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue';

interface Props {
  countryCode: string;
  phone: string;
  error?: string;
  countryCodes?: string[];
}

interface Emits {
  (e: 'update:countryCode', value: string): void;
  (e: 'update:phone', value: string): void;
  (e: 'update', value: { countryCode: string, phone: string }): void;
}

const props = withDefaults(defineProps<Props>(), {
  error: '',
  countryCodes: () => ['+886', '+1']
});

const emit = defineEmits<Emits>();

const modelCountryCode = ref(props.countryCode);
const modelPhone = ref(props.phone);

watch(() => props.countryCode, (newVal) => {
  modelCountryCode.value = newVal;
});

watch(() => props.phone, (newVal) => {
  modelPhone.value = newVal;
});

// 發送更新事件
const emitUpdate = () => {
  emit('update:countryCode', modelCountryCode.value);
  emit('update:phone', modelPhone.value);
  emit('update', {
    countryCode: modelCountryCode.value,
    phone: modelPhone.value
  });
};
</script>

<style lang="scss" scoped>
@use "@/assets/styles/forms.scss";

.phone-input-container {
  .country-code {
    width: 100px;
    padding: 12px;
    border: 1px solid #444;
    border-radius: 8px 0 0 8px;
    border-right: none;
    background-color: #333;
    color: white;
  }

  .form-input {
    border-radius: 0 8px 8px 0;
  }
}
</style>
