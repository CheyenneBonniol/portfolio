<template>
    <div class="project-item" :class="{ open: isOpen }">
        <button class="project-header" @click="$emit('toggle')">
            <div class="header-left">
                <div>
                    <p class="year">{{ project.annee }}</p>
                    <h2 class="nom">{{ project.nom }}</h2>
                    <div class="titre">{{ project.titre }}</div>
                </div>
            </div>
            <div class="header-right">
                <span class="project-type">
                    {{ project.type }}
                </span>
                <div class="badges">
                    <span v-for="competence in project.competences" :key="competence" class="badge" :style="{
                        background: competenceColor(competence) + '18',
                        color: competenceColor(competence),
                        border: '1px solid ' + competenceColor(competence) + '44'
                    }">
                        {{ competence }}
                    </span>
                </div>
            </div>
        </button>

        <Transition name="expand">
            <div v-if="isOpen" class="project-body">
                <div class="body-grid">
                    <div>
                        <p class="body-label">Description</p>
                        <p class="body-text">{{ project.descriptionLongue }}</p>

                        <p class="body-label">Apprentissages critiques</p>
                        <ul class="ac-list">
                            <li v-for="ac in project.apprentissages" :key="ac.code" class="ac-row">
                                <code class="ac-code">{{ ac.code }}</code>
                                <span class="body-text">{{ ac.description }}</span>
                            </li>
                        </ul>

                        <div v-if="project.competencesDeveloppees?.length">
                            <p class="body-label">Compétences développées</p>
                            <div class="tag-list">
                                <span v-for="s in project.competencesDeveloppees" :key="s" class="tag">{{ s }}</span>
                            </div>
                        </div>
                    </div>

                    <div>
                        <p class="body-label">Technologies</p>
                        <div class="tech-list">
                            <div v-for="tech in techs" :key="tech!.id" class="tech-row">
                                <div class="tech-info">
                                    <strong>{{ tech!.nom }}</strong>
                                    <span style="color:var(--text-muted); font-size:12px;">{{ tech!.categorie }}</span>
                                </div>
                                <div class="tech-dots">
                                    <span v-for="i in 5" :key="i" class="dot"
                                        :style="i <= tech!.niveau ? { background: tech!.couleur } : {}"></span>
                                </div>
                            </div>
                        </div>
                        <div class="project-links">
                            <a v-if="project.github" :href="project.github" target="_blank" class="btn preuve-btn">
                                GitHub
                            </a>

                            <a v-if="project.website" :href="project.website" target="_blank" class="btn preuve-btn">
                                Voir le site
                            </a>
                        </div>
                        <div v-if="project.captures?.length" class="captures-section">
                            <p class="body-label">Captures</p>

                            <div class="captures-grid">
                                <img v-for="capture in project.captures" :key="capture" :src="capture"
                                    class="capture-thumb" @click="selectedImage = capture">
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </Transition>
    </div>
    <Teleport to="body">
        <div v-if="selectedImage" class="lightbox" @click="selectedImage = null">
            <img :src="selectedImage" class="lightbox-image" @click.stop>
        </div>
    </Teleport>
</template>
<script setup lang="ts">
import { computed, ref } from 'vue'
import technologiesData from '../data/technologies.json'
import competencesData from '../data/competences.json'
const selectedImage = ref<string | null>(null)
const props = defineProps<{
    project: {
        id: number
        nom: string
        titre: string
        annee: string
        importance: number

        type: 'Universitaire' | 'Personnel' | 'Professionnel'

        descriptionCourte: string
        descriptionLongue: string

        competences: string[]

        technologies: number[]

        apprentissages: {
            code: string
            description: string
        }[]

        competencesDeveloppees?: string[]
        website?: string
        github?: string

        image?: string
        captures?: string[]
    }

    index: number
    isOpen: boolean
}>()

defineEmits<{
    toggle: []
}>()

const techs = computed(() =>
    props.project.technologies
        .map(id => technologiesData.find(t => t.id === id))
        .filter(Boolean)
)

function competenceColor(name: string): string {
    return (
        competencesData.find(c => c.nom === name)?.couleur ??
        '#8b1a1a'
    )
}
</script>

<style scoped>
.captures-section {
    margin-top: 20px;
}

.captures-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 8px;
}

.capture-thumb {
    width: 100%;
    height: 80px;
    object-fit: cover;
    border-radius: 8px;
    border: 1px solid var(--border);
    cursor: pointer;
    transition: all 0.2s;
}

.capture-thumb:hover {
    transform: scale(1.03);
    border-color: var(--red);
}

/* Lightbox */

.lightbox {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.85);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 9999;
    padding: 40px;
}

.lightbox-image {
    max-width: 90vw;
    max-height: 90vh;
    border-radius: 12px;
    box-shadow: 0 0 40px rgba(0,0,0,0.5);
}

.project-type {
    font-size: 11px;
    padding: 3px 10px;
    border-radius: 999px;
    background: var(--bg-soft);
    border: 1px solid var(--border);
}

.project-item {
    border: 1px solid var(--border);
    margin-bottom: 10px;
    border-radius: 20px;
    background: var(--bg-card);
    box-shadow: 0 0 4px var(--red-border);
}

.project-item:last-child {
    border-bottom: none;
}

.project-item.open {
    background: #fff9f9;
}

.project-header {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    padding: 18px 24px;
    background: none;
    border: none;
    cursor: pointer;
    text-align: left;
    transition: background 0.2s;

}

.project-header:hover {
    background: var(--bg-soft);
    border-radius: 20px;
}

.header-left {
    display: flex;
    align-items: center;
    gap: 16px;
}

.header-right {
    display: flex;
    align-items: center;
    gap: 10px;
    flex-shrink: 0;
}

.index {
    font-size: 12px;
    font-weight: 600;
    color: var(--red);
    min-width: 22px;
}

.year {
    font-size: 11px;
    color: var(--text-muted);
    margin-bottom: 2px;
}

.nom {
    font-size: 18px;
    font-weight: 600;
    color: var(--text);
    line-height: 1.3;
}

.titre {
    font-size: 10px;
    color: gray;
}

.badge {
    font-size: 11px;
    font-weight: 500;
    padding: 3px 10px;
    border-radius: 99px;
    white-space: nowrap;
    margin: 3px 5px;
}

.chevron {
    font-size: 16px;
    color: var(--text-muted);
    transition: transform 0.2s;
    display: inline-block;
}

.chevron.up {
    transform: rotate(180deg);
    color: var(--red);
}

/* Body */
.project-body {
    overflow: hidden;
}

.body-grid {
    display: grid;
    grid-template-columns: 1fr 200px;
    gap: 32px;
    padding: 24px 24px 28px 24px;
}

.body-label {
    font-size: 11px;
    font-weight: 600;
    letter-spacing: 0.07em;
    text-transform: uppercase;
    color: var(--red);
    margin-bottom: 8px;
    margin-top: 18px;
}

.body-label:first-child {
    margin-top: 8px;
}

.body-text {
    font-size: 14px;
    color: var(--text-sub);
    line-height: 1.7;
}

.ac-list {
    list-style: none;
    display: flex;
    flex-direction: column;
    gap: 5px;
    margin-bottom: 0;
}

.ac-row {
    display: flex;
    align-items: baseline;
    gap: 10px;
    padding: 7px 10px;
    background: var(--bg-soft);
    border-radius: 6px;
    border-left: 2px solid var(--red-border);
}

.ac-code {
    font-size: 10px;
    font-weight: 700;
    color: var(--red);
    white-space: nowrap;
    flex-shrink: 0;
}

.tag-list {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
}

.tech-chips {
    display: flex;
    flex-direction: column;
    gap: 4px;
}

.tech-chip {
    font-size: 12px;
    padding: 5px 10px;
    background: var(--bg-soft);
    border: 1px solid var(--border);
    border-radius: 6px;
    color: var(--text-sub);
}

.preuve-btn {
    display: inline-block;
    margin-top: 12px;
    font-size: 13px;
    color: white;
    border: 1px solid var(--red-border);
    background-color: var(--red);
    padding: 6px 12px;
    border-radius: 6px;
    transition: all 0.2s;
}

.preuve-btn:hover {
    background: var(--red-bg);
}

/* Transition */
.expand-enter-active,
.expand-leave-active {
    transition: max-height 0.3s ease, opacity 0.2s ease;
    max-height: 800px;
    overflow: hidden;
}

.expand-enter-from,
.expand-leave-to {
    max-height: 0;
    opacity: 0;
}

@media (max-width: 700px) {
    .body-grid {
        grid-template-columns: 1fr;
        padding-left: 24px;
    }

    .badge {
        display: none;
    }
}
</style>