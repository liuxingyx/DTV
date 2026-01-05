<template>
  <div style="max-width: 1440px; margin: 0 auto; padding: 0 32px;">
    <section style="margin-bottom: 32px;">
      <div
        style="
          display: flex;
          align-items: center;
          justify-content: space-between;
          gap: 16px;
          margin-bottom: 14px;
        "
      >
        <div style="display: flex; flex-wrap: wrap; gap: 10px;">
          <button
            v-for="cat in primaryCategories"
            :key="cat.id"
            type="button"
            class="primary-chip"
            :class="{ active: activePrimaryCategory === cat.id }"
            @click="activePrimaryCategory = cat.id"
          >
            {{ cat.name }}
          </button>
        </div>

        <button
          type="button"
          class="toggle-categories"
          @click="isCategoriesExpanded = !isCategoriesExpanded"
        >
          {{ isCategoriesExpanded ? 'Show less' : 'See all' }}
          <ChevronDown :size="16" :class="{ rotated: isCategoriesExpanded }" />
        </button>
      </div>

      <div
        class="category-cloud"
        :style="{ maxHeight: isCategoriesExpanded ? '200px' : '52px' }"
      >
        <div
          v-for="cat in CATEGORIES"
          :key="cat.id"
          class="category-chip"
        >
          {{ cat.name }}
        </div>
      </div>
    </section>

    <section>
      <div class="streamer-grid">
        <div
          v-for="streamer in filteredStreamers"
          :key="streamer.id"
          class="fluent-card streamer-card"
          @click="emit('select-streamer', streamer)"
        >
          <div class="streamer-cover">
            <img :src="streamer.cover" :alt="streamer.title" />
            <div class="platform-chip">
              {{ streamer.platform.toUpperCase() }}
            </div>
            <div class="viewer-chip">
              <span class="live-dot" />
              {{ streamer.viewerCount }}
            </div>
          </div>
          <div class="streamer-body">
            <img :src="streamer.avatar" :alt="streamer.nickname" class="streamer-avatar" />
            <div style="min-width: 0; flex: 1;">
              <h3 class="streamer-title">{{ streamer.title }}</h3>
              <div class="streamer-meta">
                <p>{{ streamer.nickname }}</p>
                <span class="meta-dot" />
                <div class="viewer-meta">
                  <Users :size="14" />
                  {{ streamer.viewerCount }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue';
import { ChevronDown, Users } from 'lucide-vue-next';
import { CATEGORIES, MOCK_STREAMERS } from '../data/mock';
import type { Platform, Streamer } from '../types';

const props = defineProps<{
  activePlatform: Platform | 'all';
  searchQuery: string;
}>();

const emit = defineEmits<{
  (event: 'select-streamer', streamer: Streamer): void;
}>();

const primaryCategories = [
  { id: 'all', name: 'All' },
  { id: 'game', name: 'Gaming' },
  { id: 'entertainment', name: 'Entertainment' },
  { id: 'lifestyle', name: 'Lifestyle' },
  { id: 'knowledge', name: 'Knowledge' },
  { id: 'sports', name: 'Sports' },
];

const isCategoriesExpanded = ref(false);
const activePrimaryCategory = ref('all');

const filteredStreamers = computed(() => {
  const query = props.searchQuery.toLowerCase();
  return MOCK_STREAMERS.filter((streamer) => {
    const matchesPlatform = props.activePlatform === 'all' || streamer.platform === props.activePlatform;
    const matchesSearch = streamer.nickname.toLowerCase().includes(query)
      || streamer.title.toLowerCase().includes(query);
    return matchesPlatform && matchesSearch;
  });
});
</script>

<style scoped>
.primary-chip {
  padding: 10px 18px;
  border-radius: var(--radius-full);
  background-color: transparent;
  border: 1px solid transparent;
  cursor: pointer;
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--text-secondary);
  transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
}

.primary-chip.active {
  background-color: var(--bg-secondary);
  border-color: var(--border);
  color: var(--text-primary);
  font-weight: 700;
  box-shadow: var(--shadow-low);
}

.primary-chip:hover {
  transform: translateY(-1px);
}

.toggle-categories {
  display: flex;
  align-items: center;
  gap: 6px;
  color: var(--text-secondary);
  font-size: 0.85rem;
  font-weight: 600;
  white-space: nowrap;
  transition: color 0.2s ease, transform 0.2s ease;
}

.toggle-categories:hover {
  color: var(--accent);
  transform: translateX(-4px);
}

.toggle-categories svg {
  transition: transform 0.2s ease;
}

.toggle-categories svg.rotated {
  transform: rotate(180deg);
}

.category-cloud {
  overflow: hidden;
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  transition: max-height 0.3s cubic-bezier(0.16, 1, 0.3, 1);
}

.category-chip {
  padding: 12px 24px;
  border-radius: var(--radius-full);
  background-color: var(--bg-secondary);
  border: 1px solid var(--border);
  cursor: pointer;
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--text-primary);
  white-space: nowrap;
  box-shadow: var(--shadow-low);
  transition: background-color 0.2s ease, transform 0.2s ease;
}

.category-chip:hover {
  background-color: var(--bg-tertiary);
  transform: translateY(-2px) scale(1.05);
}

.streamer-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 20px;
}

.streamer-card {
  overflow: hidden;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  border-radius: var(--radius-xl);
  transition: transform 0.2s ease;
  will-change: transform;
}

.streamer-card:hover {
  transform: translateY(-6px) scale(1.01);
}

.streamer-cover {
  position: relative;
  aspect-ratio: 16 / 9;
  overflow: hidden;
}

.streamer-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.platform-chip {
  position: absolute;
  top: 16px;
  right: 16px;
  background-color: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(16px) saturate(200%);
  color: #fff;
  padding: 6px 12px;
  border-radius: var(--radius-md);
  font-size: 0.7rem;
  font-weight: 800;
  border: 1px solid rgba(255, 255, 255, 0.2);
  letter-spacing: 0.05em;
}

.viewer-chip {
  position: absolute;
  bottom: 16px;
  left: 16px;
  background-color: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(8px);
  color: #fff;
  padding: 6px 12px;
  border-radius: var(--radius-md);
  font-size: 0.75rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 8px;
}

.live-dot {
  width: 8px;
  height: 8px;
  background-color: #34c759;
  border-radius: 50%;
  box-shadow: 0 0 10px #34c759;
}

.streamer-body {
  padding: 20px;
  display: flex;
  gap: 16px;
}

.streamer-avatar {
  width: 48px;
  height: 48px;
  border-radius: var(--radius-md);
  object-fit: cover;
  border: 1px solid var(--border);
}

.streamer-title {
  font-size: 1rem;
  font-weight: 800;
  margin-bottom: 6px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  color: var(--text-primary);
  letter-spacing: -0.02em;
}

.streamer-meta {
  display: flex;
  align-items: center;
  gap: 10px;
}

.streamer-meta p {
  color: var(--text-secondary);
  font-size: 0.85rem;
  font-weight: 600;
}

.meta-dot {
  width: 4px;
  height: 4px;
  background-color: var(--border);
  border-radius: 50%;
}

.viewer-meta {
  display: flex;
  align-items: center;
  gap: 6px;
  color: var(--text-secondary);
  font-size: 0.8rem;
  font-weight: 500;
}
</style>
