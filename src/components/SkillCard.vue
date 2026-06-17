<template>
  <div class="skill-card" :style="{ '--skill-color': competence.couleur }">
    <div class="skill-dot"></div>
    <div class="skill-content">
      <h3 class="skill-name">{{ competence.nom }}</h3>
      <p class="skill-desc">{{ competence.description }}</p>
    </div>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  competence: {
    id: number
    nom: string
    description: string
    couleur: string
  }
}>()
</script>

<style scoped>
.skill-card {
  display: flex;
  align-items: flex-start;
  gap: 14px;
  padding: 16px 20px;
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-left: 3px solid var(--skill-color);
  border-radius: var(--radius-md);
  transition: background var(--transition);
}

.skill-card:hover {
  background: var(--bg-card-hover);
}

.skill-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--skill-color);
  flex-shrink: 0;
  margin-top: 6px;
}

.skill-name {
  font-size: 14px;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 4px;
}

.skill-desc {
  font-size: 12px;
  color: var(--text-secondary);
  line-height: 1.5;
}
</style>