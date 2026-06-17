<template>
  <main style="padding-top: 80px; padding-bottom: 64px;">
    <div class="container">

      <div style="margin-bottom: 24px;">
        <h1 style="font-size: 36px; font-weight: 700;">Projets</h1>
      </div>
      <div class="type-filter-section">
        <div class="chips">
          <button v-for="filter in typeFilters" :key="filter.value" class="chip"
            :class="{ active: selectedTypes.includes(filter.value) }" @click="toggleItem(selectedTypes, filter.value)">
            {{ filter.label }}
          </button>
        </div>
      </div>
      <!-- Barre de filtres -->
      <div class="filter-bar">

        <!-- Année -->
        <div class="filter-group">
          <span class="filter-label">Année</span>
          <div class="chips">
            <button v-for="a in ['BUT 1', 'BUT 2']" :key="a" class="chip"
              :class="{ active: selectedAnnees.includes(a) }" @click="toggleItem(selectedAnnees, a)">{{ a }}</button>
          </div>
        </div>


        <!-- Compétence -->
        <div class="filter-group">
          <span class="filter-label">Compétence</span>
          <div class="chips">
            <button v-for="c in competencesData" :key="c.nom" class="chip"
              :class="{ active: selectedCompetences.includes(c.nom) }"
              :style="selectedCompetences.includes(c.nom) ? { background: c.couleur, borderColor: c.couleur } : {}"
              @click="toggleItem(selectedCompetences, c.nom)">{{ c.nom }}</button>
          </div>
        </div>


        <!-- Technologie -->
        <div class="filter-group" ref="techDropdownRef">
          <span class="filter-label">Technologie</span>
          <div class="dropdown-wrapper">
            <button class="dropdown-trigger" :class="{ open: techOpen }" @click="techOpen = !techOpen">
              <span>{{ selectedTechs.length ? `${selectedTechs.length} sélectionnée${selectedTechs.length > 1 ? 's' :
                ''}` : 'Toutes' }}</span>
              <span class="arrow">▾</span>
            </button>
            <div v-if="techOpen" class="dropdown-panel">
              <div v-for="cat in techCategories" :key="cat" class="dropdown-group">
                <p class="dropdown-cat">{{ cat }}</p>
                <div class="dropdown-items">
                  <button v-for="t in techsByCategory(cat)" :key="t.id" class="dropdown-item"
                    :class="{ active: selectedTechs.includes(t.id) }" @click="toggleItem(selectedTechs, t.id)"> {{ t.nom
                    }}
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>

      </div>
        <button v-if="hasFilter" class="reset-btn" @click="resetFilters">✕ Effacer</button>

      <p style="font-size:13px; color:var(--text-muted); margin-bottom:12px;">
        {{ filteredProjects.length }} projet{{ filteredProjects.length !== 1 ? 's' : '' }}
      </p>

      <div class="card" style="overflow:hidden;">
        <ProjectCard v-for="(project, i) in filteredProjects" :key="project.id" :project="project" :index="i + 1"
          :is-open="openId === project.id" :resolved-techs="resolveTechs(project.technologies)"
          @toggle="toggleOpen(project.id)" />
        <div v-if="filteredProjects.length === 0" class="empty">
          Aucun projet ne correspond aux filtres sélectionnés.
        </div>
      </div>

    </div>
  </main>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import ProjectCard from '../components/ProjectCard.vue'
import projectsData from '../data/projects.json'
import competencesData from '../data/competences.json'
import technologiesData from '../data/technologies.json'

const openId = ref<number | null>(null)
const selectedAnnees = ref<string[]>([])
const selectedCompetences = ref<string[]>([])
const selectedTechs = ref<number[]>([])
const techOpen = ref(false)
const techDropdownRef = ref<HTMLElement | null>(null)

const techCategories = [...new Set(technologiesData.map(t => t.categorie))]
const techsByCategory = (cat: string) => technologiesData.filter(t => t.categorie === cat)

const hasFilter = computed(() =>
  selectedAnnees.value.length > 0 ||
  selectedTypes.value.length > 0 ||
  selectedCompetences.value.length > 0 ||
  selectedTechs.value.length > 0
)

function toggleItem<T>(arr: T[], val: T) {
  const i = arr.indexOf(val)
  i >= 0 ? arr.splice(i, 1) : arr.push(val)
}

function resetFilters() {
  selectedAnnees.value = []
  selectedCompetences.value = []
  selectedTechs.value = []
  selectedTypes.value = []
}

function onClickOutside(e: MouseEvent) {
  if (techDropdownRef.value && !techDropdownRef.value.contains(e.target as Node))
    techOpen.value = false
}
onMounted(() => document.addEventListener('click', onClickOutside))
onUnmounted(() => document.removeEventListener('click', onClickOutside))

const filteredProjects = computed(() =>
  [...projectsData]
    .filter(p => {
      // Filtre année
      if (selectedAnnees.value.length && !selectedAnnees.value.includes(p.annee))
        return false

      if (
        selectedTypes.value.length &&
        !selectedTypes.value.includes(p.type.toLowerCase())
      ) {
        return false
      }


      // Filtre compétences — p.competences est un tableau
      if (selectedCompetences.value.length) {
        const pComps: string[] = Array.isArray(p.competences) ? p.competences : [(p).competences]
        if (!selectedCompetences.value.some(c => pComps.includes(c)))
          return false
      }

      // Filtre technos
      if (selectedTechs.value.length) {
        const ids = p.technologies.map(t =>
          typeof t === 'number' ? t : technologiesData.find(td => td.nom === t)?.id
        )
        if (!selectedTechs.value.some(id => ids.includes(id)))
          return false
      }

      return true
    })
    .sort((a, b) => a.importance - b.importance)
)
const selectedTypes = ref<string[]>([])

const typeFilters = [
  { label: 'Universitaire', value: 'universitaire' },
  { label: 'Personnel', value: 'personnel' },
  { label: 'Professionnel', value: 'professionnel' }
]

function toggleOpen(id: number) { openId.value = openId.value === id ? null : id }

function resolveTechs(technologies: (number | string)[]) {
  return technologies.map(t => {
    if (typeof t === 'number') return technologiesData.find(td => td.id === t) ?? null
    return technologiesData.find(td => td.nom === t)
      ?? { id: 0, nom: t as string, couleur: '#9e7878', logo: '', categorie: '', niveau: 0, description: '' }
  }).filter(Boolean) as typeof technologiesData
}
</script>

<style scoped>
.type-filter-section {
  margin-bottom: 14px;
}

.type-filter-section .chips {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
}
.filter-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: white;
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 20px 12px;
  margin-bottom: 2px;
  box-shadow: 0 0 10px var(--red-bg);
}

.filter-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.filter-label {
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.07em;
  color: var(--red);
  white-space: nowrap;
}


.chips {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
}

.chip {
  padding: 5px 12px;
  border-radius: 99px;
  border: 1px solid var(--border);
  background: var(--bg-soft);
  color: var(--text-sub);
  font-size: 13px;
  cursor: pointer;
  transition: all 0.15s;
  white-space: nowrap;
}

.chip:hover {
  border-color: var(--red-border);
  color: var(--red);
}

.chip.active {
  background: var(--red);
  border-color: var(--red);
  color: white;
}

/* Dropdown */
.dropdown-wrapper {
  position: relative;
}

.dropdown-trigger {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 5px 12px;
  border-radius: 8px;
  border: 1px solid var(--border);
  background: var(--bg-soft);
  color: var(--text-sub);
  font-size: 13px;
  cursor: pointer;
  transition: all 0.15s;
  min-width: 120px;
  justify-content: space-between;
}

.dropdown-trigger:hover,
.dropdown-trigger.open {
  border-color: var(--red-border);
  color: var(--red);
}

.arrow {
  transition: transform 0.2s;
  font-size: 12px;
}

.dropdown-trigger.open .arrow {
  transform: rotate(180deg);
}

.dropdown-panel {
  position: absolute;
  top: calc(100% + 6px);
  left: 0;
  z-index: 200;
  background: white;
  border: 1px solid var(--border);
  border-radius: var(--radius);
  box-shadow: 0 8px 24px rgba(139, 26, 26, 0.1);
  width: 280px;
  max-height: 360px;
  overflow-y: auto;
  padding: 8px;
}

.dropdown-group {
  margin-bottom: 8px;
}

.dropdown-cat {
  font-size: 10px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--text-muted);
  padding: 4px 8px;
}

.dropdown-items {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.dropdown-item {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 8px;
  border: none;
  background: none;
  border-radius: 6px;
  font-size: 13px;
  color: var(--text-sub);
  cursor: pointer;
  text-align: left;
  transition: background 0.15s;
}

.dropdown-item:hover {
  background: var(--bg-soft);
}

.dropdown-item.active {
  color: var(--red);
  font-weight: 500;
  background: var(--red-bg);
}

.check {
  width: 14px;
  font-size: 11px;
  color: var(--red);
  flex-shrink: 0;
}

.reset-btn {
  padding: 5px 12px;
  border-radius: 99px;
  border: 1px solid var(--border);
  background: transparent;
  color: var(--text-muted);
  font-size: 12px;
  cursor: pointer;
  transition: all 0.15s;
  white-space: nowrap;
}

.reset-btn:hover {
  color: var(--red);
  border-color: var(--red-border);
}

.empty {
  padding: 48px;
  text-align: center;
  color: var(--text-muted);
  font-size: 14px;
}

.card {
  padding: 10px;
}

@media (max-width: 768px) {
  .filter-bar {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }
}
</style>