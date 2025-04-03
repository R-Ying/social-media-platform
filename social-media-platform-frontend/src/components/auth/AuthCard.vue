<template>
  <div class="auth-container">
    <div class="auth-card">
      <h1 class="title">{{ title }}</h1>
      <p v-if="subtitle" class="subtitle">{{ subtitle }}</p>
      <slot></slot>
    </div>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  title: string;
  subtitle?: string;
}>();
</script>

<style lang="scss" scoped>
@use "@/assets/styles/auth-pages.scss";
@use "@/assets/styles/typography.scss";
</style>
