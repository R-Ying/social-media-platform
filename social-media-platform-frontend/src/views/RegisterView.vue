<template>
  <AuthCard title="建立帳號" subtitle="加入我們的社群平台">
    <form @submit.prevent="handleRegister" class="form">
      <!-- 手機號碼輸入 -->
      <PhoneInput
        v-model:countryCode="countryCode"
        v-model:phone="phone"
        :error="errors.phone"
      />

      <!-- 驗證碼 -->
      <VerificationCodeInput
        v-model="verificationCode"
        :error="errors.verificationCode"
        :isCountingDown="isCountingDown"
        :countdownText="countdownText"
        @send-code="sendVerificationCode"
      />

      <!-- 密碼 -->
      <PasswordInput
        v-model="password"
        label="密碼"
        placeholder="請設定密碼"
        :error="errors.password"
        hint="密碼需包含至少8個字元，包括大小寫字母和數字"
      />

      <!-- 確認密碼 -->
      <PasswordInput
        v-model="confirmPassword"
        label="確認密碼"
        placeholder="請再次輸入密碼"
        :error="errors.confirmPassword"
        id="confirmPassword"
      />

      <!-- 註冊按鈕 -->
      <button type="submit" class="primary-btn" :disabled="isRegistering">
        {{ isRegistering ? '註冊中...' : '註冊' }}
      </button>
    </form>

    <!-- 已有帳號導向登入頁 -->
    <div class="auth-link">
      已有帳號？<router-link to="/login" class="link-text">登入</router-link>
    </div>
  </AuthCard>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import { useRouter } from 'vue-router';
import AuthCard from '@/components/auth/AuthCard.vue';
import PhoneInput from '@/components/auth/PhoneInput.vue';
import PasswordInput from '@/components/auth/PasswordInput.vue';
import VerificationCodeInput from '@/components/auth/VerificationCodeInput.vue';

const router = useRouter();

// 表單數據
const countryCode = ref('+886');
const phone = ref('');
const verificationCode = ref('');
const password = ref('');
const confirmPassword = ref('');

// 表單狀態
const isRegistering = ref(false);
const isCountingDown = ref(false);
const countdown = ref(60);
const countdownText = ref('獲取驗證碼');

// 錯誤訊息
const errors = reactive({
  phone: '',
  verificationCode: '',
  password: '',
  confirmPassword: ''
});

// 發送驗證碼
const sendVerificationCode = () => {
  // 驗證手機號碼格式
  if (!validatePhone()) {
    return;
  }

  isCountingDown.value = true;
  countdown.value = 60;
  countdownText.value = `重新發送(${countdown.value}s)`;

  // 模擬發送驗證碼
  console.log(`向 ${countryCode.value}${phone.value} 發送驗證碼`);

  // 倒數計時
  const timer = setInterval(() => {
    countdown.value--;
    countdownText.value = `重新發送(${countdown.value}s)`;

    if (countdown.value <= 0) {
      clearInterval(timer);
      isCountingDown.value = false;
      countdownText.value = '獲取驗證碼';
    }
  }, 1000);
};

// 驗證手機號碼
const validatePhone = (): boolean => {
  // 清除之前的錯誤
  errors.phone = '';

  // 簡單驗證手機號格式
  const phonePattern = /^\d{9,10}$/;
  if (!phone.value) {
    errors.phone = '請輸入手機號碼';
    return false;
  } else if (!phonePattern.test(phone.value)) {
    errors.phone = '請輸入有效的手機號碼';
    return false;
  }

  return true;
};

// 驗證整個表單
const validateForm = (): boolean => {
  let isValid = true;

  // 驗證手機號碼
  if (!validatePhone()) {
    isValid = false;
  }

  // 驗證碼
  if (!verificationCode.value) {
    errors.verificationCode = '請輸入驗證碼';
    isValid = false;
  } else if (verificationCode.value.length !== 6) {
    errors.verificationCode = '驗證碼應為6位數字';
    isValid = false;
  } else {
    errors.verificationCode = '';
  }

  // 密碼
  const passwordPattern = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d]{8,}$/;
  if (!password.value) {
    errors.password = '請設定密碼';
    isValid = false;
  } else if (!passwordPattern.test(password.value)) {
    errors.password = '密碼需包含至少8個字元，包括大小寫字母和數字';
    isValid = false;
  } else {
    errors.password = '';
  }

  // 確認密碼
  if (!confirmPassword.value) {
    errors.confirmPassword = '請再次輸入密碼';
    isValid = false;
  } else if (confirmPassword.value !== password.value) {
    errors.confirmPassword = '兩次密碼輸入不一致';
    isValid = false;
  } else {
    errors.confirmPassword = '';
  }

  return isValid;
};

// 註冊處理
const handleRegister = () => {
  if (!validateForm()) {
    return;
  }

  isRegistering.value = true;

  // 模擬註冊過程
  setTimeout(() => {
    console.log('註冊成功', {
      phone: `${countryCode.value}${phone.value}`,
      password: password.value
    });

    isRegistering.value = false;

    // 註冊成功，導向登入頁或首頁
    router.push('/login');
  }, 1500);
};
</script>

<style lang="scss" scoped>
@use "@/assets/styles/buttons.scss";

.form {
  margin-bottom: 24px;
}

.auth-link {
  text-align: center;
  font-size: 14px;
  color: #666;
}

.link-text {
  color: #4a90e2;
  text-decoration: none;
  font-weight: 500;
}
</style>
