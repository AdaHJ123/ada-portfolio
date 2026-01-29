<script setup>
import { useRouter } from 'vue-router'
import ProjectCard from './ProjectCard.vue'
import projects from '../data/projects'

const router = useRouter()
const goDetail = (slug) => {
  router.push(`/projects/${slug}`)
}
</script>

<template>
  <section id="projects">
    <h2 class="section-title" style="font-size:2rem; margin-bottom:16px;">Projects</h2>

    <!-- 网格布局 -->
    <div class="grid">
      <ProjectCard
        v-for="p in projects"
        :key="p.slug"
        v-bind="p"
        @click="goDetail(p.slug)"
      />
      <!-- 用 p.slug 做 key，并确保你的项目数据每条都有 slug -->
    </div>

    <div class="more-link-row">
      <router-link class="more-link" to="/works">
        View More Works
      </router-link>
    </div>

  </section>
</template>

<style scoped>

/* 容器宽度：不贴边，居中 */
#projects {
  /* 你也可以放到外层 main/App 里，这里演示就地设置 */
  max-width: 1280px;                         /* 宽屏两列更舒适 */
  margin: 0 auto;
  padding: 0 clamp(16px, 4vw, 32px);         /* 左右留边 根据屏幕尺寸来决定留边的距离*/
}


/* 一列/两列 */
.grid {
  display: grid;
  gap: 24px;
  grid-template-columns: 1fr;
  align-items: stretch;                    /* 卡片等高拉伸 */
}
/* ≥900px 时两列；也可以用 768/1024，按喜好 */
@media (min-width: 900px) {
  .grid {
     grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}


.more-link-row {
  display: flex;
  justify-content: flex-end;
  margin: 32px 0 64px; /* top margin from grid, bottom space before Skills */
}

.more-link {
  color: #e2e1e1; 
  font-weight: 700; 
  font-size: 20px;
  text-decoration: underline;
  text-underline-offset: 4px;
  transition: color 0.15s ease; /* 平滑变色效果 */
}

.more-link:hover {
  color: #9b9eaa;
}
</style>

