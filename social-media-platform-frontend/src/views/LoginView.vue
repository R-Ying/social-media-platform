<template>
  <AuthCard title="帳號登入" subtitle="歡迎回到我們的社群平台">
    <form @submit.prevent="handleLogin" class="form">
      <!-- 手機號碼輸入 -->
      <PhoneInput
        v-model:countryCode="countryCode"
        v-model:phone="phone"
        :error="errors.phone"
      />

      <!-- 登入方式選擇器 -->
      <div class="login-type-selector">
        <div
          class="login-type-option"
          :class="{ 'active': loginType === 'password' }"
          @click="setLoginType('password')"
        >
          密碼登入
        </div>
        <div
          class="login-type-option"
          :class="{ 'active': loginType === 'verification' }"
          @click="setLoginType('verification')"
        >
          驗證碼登入
        </div>
      </div>

      <!-- 根據登入類型顯示不同的輸入框 -->
      <template v-if="loginType === 'password'">
        <!-- 密碼輸入 -->
        <PasswordInput
          v-model="password"
          :error="errors.password"
          hint="密碼需包含至少8個字元"
        />
      </template>

      <template v-else>
        <!-- 驗證碼輸入 -->
        <VerificationCodeInput
          v-model="verificationCode"
          :error="errors.verificationCode"
          :isCountingDown="isCountingDown"
          :countdownText="countdownText"
          @send-code="sendVerificationCode"
        />
      </template>

      <!-- 登入按鈕 -->
      <button type="submit" class="primary-btn" :disabled="isLoggingIn">
        {{ isLoggingIn ? '登入中...' : '登入' }}
      </button>
    </form>

    <!-- 註冊帳號連結 -->
    <div class="auth-link">
      還沒有帳號？<router-link to="/register" class="link-text">立即註冊</router-link>
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
const password = ref('');
const verificationCode = ref('');
const loginType = ref('password');

// 表單狀態
const isLoggingIn = ref(false);
const isCountingDown = ref(false);
const countdown = ref(60);
const countdownText = ref('獲取驗證碼');

// 錯誤訊息
const errors = reactive({
  phone: '',
  password: '',
  verificationCode: ''
});

// 設置登入類型
const setLoginType = (type: 'password' | 'verification') => {
  loginType.value = type;
  // 切換時清除錯誤訊息
  errors.password = '';
  errors.verificationCode = '';
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

// 驗證密碼
const validatePassword = (): boolean => {
  // 清除之前的錯誤
  errors.password = '';

  if (!password.value) {
    errors.password = '請輸入密碼';
    return false;
  } else if (password.value.length < 8) {
    errors.password = '密碼需包含至少8個字元';
    return false;
  }

  return true;
};

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

// 驗證表單
const validateForm = (): boolean => {
  let isValid = true;

  // 驗證手機號碼
  if (!validatePhone()) {
    isValid = false;
  }

  // 根據登入類型驗證不同欄位
  if (loginType.value === 'password') {
    // 驗證密碼
    if (!validatePassword()) {
      isValid = false;
    }
  } else {
    // 驗證驗證碼
    if (!verificationCode.value) {
      errors.verificationCode = '請輸入驗證碼';
      isValid = false;
    } else if (verificationCode.value.length !== 6) {
      errors.verificationCode = '驗證碼應為6位數字';
      isValid = false;
    } else {
      errors.verificationCode = '';
    }
  }

  return isValid;
};

// 登入處理
const handleLogin = () => {
  if (!validateForm()) {
    return;
  }

  isLoggingIn.value = true;

  // 根據登入類型準備不同的登入數據
  const loginData = {
    phone: `${countryCode.value}${phone.value}`,
    loginType: loginType.value,
  };

  // 根據登入類型添加不同的身份驗證資訊
  if (loginType.value === 'password') {
    Object.assign(loginData, { password: password.value });
  } else {
    Object.assign(loginData, { verificationCode: verificationCode.value });
  }

  // 模擬登入過程
  setTimeout(() => {
    console.log('登入成功', loginData);

    isLoggingIn.value = false;

    // 登入成功，導向首頁
    router.push('/home');
  }, 1500);
};
</script>

<style lang="scss" scoped>
@use "@/assets/styles/buttons.scss";

.form {
  margin-bottom: 24px;
}

.login-type-selector {
  display: flex;
  margin-bottom: 20px;
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid #444;
}

.login-type-option {
  flex: 1;
  text-align: center;
  padding: 12px;
  background-color: #333;
  cursor: pointer;
  transition: background-color 0.3s, color 0.3s;

  &.active {
    background-color: #4a90e2;
    color: white;
  }
}

.forget-password {
  text-align: right;
  margin-bottom: 20px;
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
