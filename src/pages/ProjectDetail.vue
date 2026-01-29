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

    <div v-for="section in project.sections" :key="section.id || section.title" class="part">
      <h3 v-if="section.title" class="part-header">{{ section.title }}</h3>
      <div v-if="section.text" class="project-detail-text" v-html="section.text"></div>
      <div
        v-if="section.images && section.images.length"
        :class="['project-detail-gallery', { 'project-detail-gallery--stack': section.layout === 'stack' }]"
        :data-section="section.dataSection || null"
      >
        <!-- 缩略图支持三种可选属性：
             thumbFit: "contain"     显示完整内容不裁切
             thumbSize: "narrow"     缩小该图占位（适合竖图/示意图）
             thumbRatio: "auto"      取消固定比例，避免上下留白 -->
        <figure
          v-for="image in section.images"
          :key="image.src"
          :class="['project-detail-item', { 'project-detail-item--narrow': image.thumbSize === 'narrow' }]"
        >
          <button
            :class="[
              'project-detail-thumb',
              {
                'project-detail-thumb--contain': image.thumbFit === 'contain',
                'project-detail-thumb--auto': image.thumbRatio === 'auto',
              },
            ]"
            type="button"
            @click="openLightbox(image)"
          >
            <img :src="image.src" :alt="image.alt || project.title" loading="lazy" />
          </button>
          <figcaption class="project-detail-caption">
            {{ image.alt || project.title }}
          </figcaption>
        </figure>
      </div>
      <p v-if="section.afterText" class="project-detail-text" v-html="section.afterText"></p>
      <div v-if="section.afterText2" class="project-detail-text" v-html="section.afterText2"></div>
      <div v-if="section.youtubeId" class="project-detail-video">
        <div class="project-detail-video-frame">
          <iframe
            :src="`https://www.youtube.com/embed/${section.youtubeId}?rel=0`"
            :title="section.videoCaption || section.title || 'Project video'"
            frameborder="0"
            allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
            allowfullscreen
          ></iframe>
        </div>
        <p v-if="section.videoCaption" class="project-detail-caption">
          {{ section.videoCaption }}
        </p>
      </div>
    </div>

    <div
      v-if="lightboxImage"
      class="lightbox"
      role="dialog"
      aria-modal="true"
      @click.self="closeLightbox"
    >
      <button class="lightbox-close" type="button" aria-label="Close image" @click="closeLightbox">
        ×
      </button>
      <img :src="lightboxImage.src" :alt="lightboxImage.alt || project.title" />
    </div>
  </div>
</template>

<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from "vue"
import { useRoute, useRouter } from "vue-router"
import projects from "../data/projects"

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
  sections: [],
}

const project = computed(() => {
  const found = projects.find((p) => p.slug === route.params.slug)
  if (!found) return defaults
  const fallbackSections = [
    {
      id: "introduction",
      title: "Introduction",
      text: found.descriptionLong || found.summary || "",
      layout: "grid",
      images: found.introImages || [],
    },
  ]
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
    sections: found.sections && found.sections.length ? found.sections : fallbackSections,
  }
})

const lightboxImage = ref(null)

const openLightbox = (image) => {
  lightboxImage.value = image
  document.body.style.overflow = "hidden"
}

const closeLightbox = () => {
  lightboxImage.value = null
  document.body.style.overflow = ""
}

const onKeydown = (event) => {
  if (event.key === "Escape" && lightboxImage.value) {
    closeLightbox()
  }
}

onMounted(() => {
  window.scrollTo({ top: 0, behavior: "auto" })
  window.addEventListener("keydown", onKeydown)
})

onBeforeUnmount(() => {
  window.removeEventListener("keydown", onKeydown)
  document.body.style.overflow = ""
})

if (!project.value.slug) router.replace("/")
</script>

<style src="../project-detail-style.css"></style>




