<script setup lang="ts">
import { RouterView } from 'vue-router'
</script>

<template>
  <RouterView />
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
  background-color: #1e1e1e;
  color: white;
  font-family: 'Arial', sans-serif;
  overflow-x: hidden;
}

#app {
  width: 100%;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* 通用按鈕樣式 */
button {
  cursor: pointer;
}

/* 通用連結樣式 */
a {
  color: #4b70e2;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

/* 通用輸入框樣式 */
input, textarea, select {
  background-color: #333;
  color: white;
  border: 1px solid #444;
  padding: 10px;
  border-radius: 4px;
  font-family: inherit;
}

input:focus, textarea:focus, select:focus {
  outline: none;
  border-color: #4b70e2;
}

/* 佔位符文字顏色 */
::placeholder {
  color: #888;
}
</style>ㄋ
