<template>
  <div class="content project-detail">
    <div class="portfolio-header">
      <router-link class="portfolio-close" to="/">
        <span class="close-icon" aria-hidden="true">
          <svg viewBox="0 0 24 24" role="presentation" focusable="false">
            <path
              d="M19 12H5M12 19l-7-7 7-7"
              stroke="currentColor"
              stroke-width="4"
              stroke-linecap="round"
              stroke-linejoin="round"
              fill="none"
            />
          </svg>
        </span>
        <small>Go Back</small>
      </router-link>
    </div>

    <div class="portfolio-top">
      <div class="content-placeholder">
        <iframe
          v-if="project.youtubeId"
          :src="`https://www.youtube.com/embed/${project.youtubeId}?rel=0`"
          :title="project.title"
          frameborder="0"
          allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
          allowfullscreen
        ></iframe>
        <video v-else playsinline loop muted autoplay preload="metadata">
          <source v-if="project.videoMp4" :src="project.videoMp4" type="video/mp4" />
          <source v-if="project.videoWebm" :src="project.videoWebm" type="video/webm" />
        </video>
      </div>
      <h2>{{ project.title }}</h2>
    </div>

    <div class="experience">
      <section class="job-card">
        <h3>About</h3>
        <p>{{ project.summary || "More details coming soon." }}</p>
      </section>

      <section class="job-card">
        <h3>Project Info</h3>
        <ul>
          <li v-if="project.role"><span class="list-label">Role:</span> {{ project.role }}</li>
          <li v-if="project.duration"><span class="list-label">Time:</span> {{ project.duration }}</li>
          <li v-if="project.year"><span class="list-label">Year:</span> {{ project.year }}</li>
          <li v-if="project.tags && project.tags.length">
            <span class="list-label">Tech:</span> {{ project.tags.join(', ') }}
          </li>
        </ul>
        <div class="project-links" v-if="project.links && (project.links.demo || project.links.github)">
          <a
            v-if="project.links.demo"
            class="url-link"
            :href="project.links.demo"
            target="_blank"
            rel="noreferrer"
          >
            Demo
          </a>
          <a
            v-if="project.links.github"
            class="url-link"
            :href="project.links.github"
            target="_blank"
            rel="noreferrer"
          >
            GitHub
          </a>
        </div>
      </section>
    </div>

    <div class="part">
      <h3 class="part-header">Introduction</h3>
      <p>{{ project.descriptionLong || project.summary || "More details coming soon." }}</p>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted } from "vue"
import { useRoute, useRouter } from "vue-router"
import projects from "../data/projects.json"

const route = useRoute()
const router = useRouter()

const defaults = {
  slug: "",
  title: "",
  role: "",
  duration: "",
  tags: [],
  summary: "",
  descriptionLong: "",
  videoMp4: "",
  videoWebm: "",
  year: "",
  youtubeId: "",
  links: {},
}

const project = computed(() => {
  const found = projects.find((p) => p.slug === route.params.slug)
  if (!found) return defaults
  return {
    ...defaults,
    ...found,
    duration: found.duration || found.timeframe || "",
    tags: found.tags || found.tech || [],
    summary: found.summary || found.description || "",
    videoMp4: found.videoMp4 || found.videoSrc || "",
    videoWebm: found.videoWebm || "",
    youtubeId: found.youtubeId || "",
    links: found.links || {},
  }
})

onMounted(() => {
  window.scrollTo({ top: 0, behavior: "auto" })
})

if (!project.value.slug) router.replace("/")
</script>



