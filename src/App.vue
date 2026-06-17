<template>
  <div id="app">
    <NavBar />
    <RouterView />
  </div>
</template>

<script setup lang="ts">
import NavBar from './components/NavBar.vue'
import { RouterView } from 'vue-router'
</script>

<style>
@import './assets/main.css';
</style>