<template>
  <nav class="navbar" :class="{ scrolled: isScrolled }">
    <div class="container nav-inner">
      <RouterLink to="/" class="brand">Cheyenne<span>.</span></RouterLink>
      <div class="links">
        <RouterLink to="/" exact-active-class="active" class="link">Accueil</RouterLink>
        <RouterLink to="/projects" active-class="active" class="link">Projets</RouterLink>
      </div>
    </div>
  </nav>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { RouterLink } from 'vue-router'
const isScrolled = ref(false)
function onScroll() { isScrolled.value = window.scrollY > 20 }
onMounted(() => window.addEventListener('scroll', onScroll))
onUnmounted(() => window.removeEventListener('scroll', onScroll))
</script>

<style scoped>
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 100;
  padding: 16px 0;
  transition: all 0.2s ease;
  box-shadow: 0 0 5px var(--red);
}

.navbar.scrolled {
  background: rgba(250, 247, 247, 0.95);
  backdrop-filter: blur(8px);
  border-bottom: 1px solid var(--border);
  padding: 10px 0;
}

.nav-inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.brand {
  font-size: 20px;
  font-weight: 700;
  color: var(--text);
}

.brand span {
  color: var(--red);
}

.links {
  display: flex;
  gap: 4px;
}

.link {
  padding: 6px 14px;
  border-radius: 8px;
  font-size: 14px;
  color: var(--text-sub);
  transition: all 0.2s;
}

.link:hover {
  background: var(--bg-soft);
  color: var(--text);
}

.link.active {
  background: var(--red-bg);
  color: var(--red);
  font-weight: 600;
}
</style>