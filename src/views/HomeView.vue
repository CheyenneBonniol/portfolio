<template>
  <main style="padding-top: 80px;">

    <!-- Hero : profil + infos côte à côte sur toute la largeur -->
    <section class="container" style="padding-top: 48px; padding-bottom: 64px;">
      <div class="hero">

        <!-- Colonne gauche : carte profil -->
        <aside class="profile-card cardAccueil">

          <div class="profile-body">

            <h1 class="profile-name">{{ profile.nom }}</h1>
            <p class="profile-role">{{ profile.formation }} · {{ profile.annee }}</p>
            <p class="profile-city">📍 {{ profile.ville }}</p>
          </div>

          <div class="profile-actions">
            <a :href="profile.cv" download class="btn btn-primary" style="width:100%; justify-content:center;">
              Télécharger mon CV
            </a>
            <div style="display:flex; gap:8px;">
              <a :href="profile.linkedin" target="_blank" class="btn btn-outline"
                style="flex:1; justify-content:center;">LinkedIn</a>
              <a :href="profile.github" target="_blank" class="btn btn-outline"
                style="flex:1; justify-content:center;">GitHub</a>
            </div>
            <a :href="`mailto:${profile.email}`" class="email-link">{{ profile.email }}</a>
          </div>
        </aside>

        <!-- Colonne droite : infos -->
        <div class="profile-content">

          <div class="section">
            <span class="label">À propos</span>
            <p style="color: var(--text-sub); line-height: 1.75;">{{ profile.description }}</p>
          </div>

          <div class="section">
            <span class="label">Centres d'intérêt</span>
            <div class="tag-list">
              <span v-for="p in profile.passions" :key="p" class="tag">{{ p }}</span>
            </div>
          </div>

          <div class="section">
            <span class="label">Technologies favorites</span>
            <div class="tech-list">
              <div v-for="tech in favoriteTechs" :key="tech.id" class="tech-row">
                <div class="tech-info">
                  <strong>{{ tech.nom }}</strong>
                  <span style="color:var(--text-muted); font-size:12px;">{{ tech.categorie }}</span>
                </div>
                <div class="tech-dots">
                  <span v-for="i in 5" :key="i" class="dot"
                    :style="i <= tech.niveau ? { background: tech.couleur } : {}"></span>
                </div>
              </div>
            </div>
          </div>

        </div>
      </div>
    </section>

    <!-- Aperçu projets -->
    <section style="border-top: 1px solid var(--border); padding: 56px 0;">
      <div class="container">
        <div class="section-header">
          <div>
            <span class="label">Réalisations</span>
            <h2 class="section-title">Projets SAE</h2>
          </div>
          <RouterLink to="/projects" class="see-all">Voir tous </RouterLink>
        </div>

        <div class="projects-preview">
          <div v-for="p in topProjects" :key="p.id" class="cardAccueil project-card" @click="$router.push('/projects')">
            <div class="project-header-preview">
              <div style="display:flex; gap:4px; flex-wrap:wrap;">
                <span v-for="competence in p.competences" :key="competence" class="tag" :style="{
                  borderColor: competenceColor(competence),
                  color: competenceColor(competence)
                }">
                  {{ competence }}
                </span>
              </div>

              <span style="font-size:11px; color:var(--text-muted);">
                {{ p.annee }}
              </span>
            </div>
            <h3 style="font-size:15px; font-weight:600; margin-bottom:5px; line-height:1.3;">
              {{ p.nom }}
            </h3>
            <div style="font-size:10px; color:gray; margin-bottom: 8px;">{{ p.titre }}</div>

            <span class="project-type">
              {{ p.type }}
            </span>
            <p style="font-size:13px; color:var(--text-sub); flex:1;">{{ p.descriptionCourte }}</p>
            <div class="mini-techs">
              <span v-for="tech in getTechnologyNames(p.technologies).slice(0, 3)" :key="tech" class="mini-tech">
                {{ tech }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </section>

  </main>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { RouterLink } from 'vue-router'
import profileData from '../data/profile.json'
import technologiesData from '../data/technologies.json'
import projectsData from '../data/projects.json'
import competencesData from '../data/competences.json'

const profile = profileData

function competenceColor(name: string) {
  return (
    competencesData.find(c => c.nom === name)?.couleur ??
    '#8b1a1a'
  )
}

const favoriteTechs = computed(() =>
  profile.technologiesFavorites
    .map(id => technologiesData.find(t => t.id === id))
    .filter(Boolean) as typeof technologiesData
)
const getTechnologyNames = (technologyIds: number[]) => {
  return technologyIds
    .map(id => technologiesData.find(t => t.id === id)?.nom)
    .filter(Boolean)
}
const topProjects = computed(() =>
  [...projectsData].sort((a, b) => a.importance - b.importance).slice(0, 3)
)
</script>

<style scoped>
.project-header-preview {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 8px;
  margin-bottom: 10px;
}

.project-header-preview>div:first-child {
  display: flex;
  gap: 4px;
  flex-wrap: wrap;
  flex: 1;
}

.project-header-preview span:last-child {
  flex-shrink: 0;
  white-space: nowrap;
}

.project-type {
  display: inline-block;
  width: min-content;
  margin-bottom: 10px;
  padding: 2px 8px;
  font-size: 11px;
  border-radius: 999px;
  background: var(--bg-soft);
  border: 1px solid var(--border);
  color: var(--text-muted);
}

/* Hero 2 colonnes pleine largeur */
.hero {
  display: grid;
  grid-template-columns: 280px 1fr;
  gap: 40px;
  align-items: start;
}

/* Carte profil */
.profile-card {
  overflow: hidden;
  position: sticky;
  top: 80px;
  border: 2px solid var(--red);
  box-shadow: 0 0 10px var(--red);
}


.profile-body {
  padding: 28px 24px 0;
  text-align: center;
}


.profile-name {
  font-size: 22px;
  font-weight: 700;
  margin-bottom: 6px;
}

.profile-role {
  font-size: 13px;
  color: var(--text-sub);
  margin-bottom: 6px;
}

.profile-city {
  font-size: 12px;
  color: var(--text-muted);
}

.profile-actions {
  padding: 20px 24px 24px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.email-link {
  text-align: center;
  font-size: 12px;
  color: var(--text-muted);
  border-top: 1px solid var(--border);
  padding-top: 10px;
  transition: color 0.2s;
}

.email-link:hover {
  color: var(--red);
}

/* Contenu droite */
.profile-content {
  display: flex;
  flex-direction: column;
  gap: 32px;
  padding-top: 4px;
}

.section {
  display: flex;
  flex-direction: column;
}

.tag-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

/* Section header */
.section-header {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  margin-bottom: 24px;
}

.section-title {
  font-size: 26px;
  font-weight: 700;
  margin-top: 4px;
}

.see-all {
  font-size: 14px;
  color: var(--red);
  font-weight: 500;
}

.see-all:hover {
  text-decoration: underline;
}

/* Projets preview */
.projects-preview {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.project-card {
  padding: 18px;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  gap: 0;
  transition: all 0.2s;
}

.project-card:hover {
  border-color: var(--red-border);
  box-shadow: 0 4px 20px rgba(139, 26, 26, 0.1);
  transform: translateY(-2px);
}

.mini-techs {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
  margin-top: 12px;
}

.mini-tech {
  font-size: 11px;
  padding: 2px 8px;
  background: var(--bg-soft);
  border: 1px solid var(--border);
  border-radius: 4px;
  color: var(--text-muted);
}

/* Responsive */
@media (max-width: 768px) {
  .hero {
    grid-template-columns: 1fr;
  }

  .profile-card {
    position: static;
  }

  .projects-preview {
    grid-template-columns: 1fr;
  }
}
</style>