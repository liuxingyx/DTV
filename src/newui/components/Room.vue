<template>
  <div
    style="display: flex; gap: 24px; height: 100%; min-height: 0;"
  >
    <div style="flex: 1; display: flex; flex-direction: column; min-width: 0; gap: 20px;">
      <div class="player-shell">
        <button
          type="button"
          class="back-btn"
          @click="emit('back')"
        >
          <ArrowLeft :size="20" />
        </button>

        <img :src="streamer.cover" alt="stream cover" class="player-cover" />

        <div class="player-status">
          <div class="player-pulse">
            <span class="player-dot" />
          </div>
          <span class="player-status-text">Secure Stream Linked</span>
        </div>

        <div class="player-controls">
          <div style="display: flex; align-items: center; gap: 20px; color: #fff;">
            <Volume2 :size="20" />
            <div class="volume-track">
              <div class="volume-level" />
            </div>
          </div>
          <div style="display: flex; align-items: center; gap: 24px; color: #fff;">
            <Settings :size="20" />
            <Maximize :size="20" />
          </div>
        </div>
      </div>

      <div class="mica info-bar">
        <div style="display: flex; gap: 20px; align-items: center;">
          <img :src="streamer.avatar" :alt="streamer.nickname" class="info-avatar" />
          <div>
            <h2 class="info-title">{{ streamer.title }}</h2>
            <div class="info-meta">
              <p>{{ streamer.nickname }}</p>
              <span class="meta-dot" />
              <div class="viewer-meta">
                <Users :size="16" />
                {{ streamer.viewerCount }}
              </div>
              <span class="platform-tag">
                {{ streamer.platform }}
              </span>
            </div>
          </div>
        </div>
        <div style="display: flex; gap: 12px;">
          <button type="button" class="follow-btn">Follow</button>
          <button type="button" class="share-btn">
            <Share2 :size="24" />
          </button>
        </div>
      </div>
    </div>

    <div class="mica chat-panel">
      <div class="chat-header">Live Interaction</div>

      <div ref="scrollRef" class="chat-messages">
        <div v-for="item in danmakuList" :key="item.id" class="chat-message">
          <span
            :style="{
              color: item.user === 'You' ? 'var(--accent)' : 'var(--text-secondary)',
            }"
            class="chat-user"
          >
            {{ item.user }}
          </span>
          <span class="chat-text">{{ item.content }}</span>
        </div>
      </div>

      <div class="chat-input-wrap">
        <div class="chat-input-shell" :class="{ focused: isChatFocused }">
          <input
            type="text"
            placeholder="Type a message..."
            v-model="inputText"
            @keydown.enter="handleSend"
            @focus="isChatFocused = true"
            @blur="isChatFocused = false"
          />
          <button type="button" @click="handleSend">
            <Send :size="20" />
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { nextTick, onBeforeUnmount, onMounted, ref, watch } from 'vue';
import { ArrowLeft, Maximize, Send, Settings, Share2, Users, Volume2 } from 'lucide-vue-next';
import type { Streamer } from '../types';

interface Danmaku {
  id: number;
  user: string;
  content: string;
  color?: string;
}

defineProps<{
  streamer: Streamer;
  theme: 'light' | 'dark';
}>();

const emit = defineEmits<{
  (event: 'back'): void;
}>();

const danmakuList = ref<Danmaku[]>([]);
const inputText = ref('');
const isChatFocused = ref(false);
const scrollRef = ref<HTMLDivElement | null>(null);
let intervalId: number | undefined;

onMounted(() => {
  const fakeUsers = ['User_01', 'Apple_Fan', 'Fluent_Pro', 'Design_Guru', 'Sky_Walker', 'Cyber_Mica'];
  const fakeMsgs = [
    'This hybrid UI looks amazing!',
    'So smooth and professional',
    'Perfect blend of platforms',
    'Quality is top notch',
    'Windows + Mac vibes!',
    'Keep it up',
  ];

  intervalId = window.setInterval(() => {
    if (danmakuList.value.length > 50) {
      return;
    }

    const newDanmaku: Danmaku = {
      id: Date.now() + Math.random(),
      user: fakeUsers[Math.floor(Math.random() * fakeUsers.length)],
      content: fakeMsgs[Math.floor(Math.random() * fakeMsgs.length)],
      color: 'var(--text-secondary)',
    };

    danmakuList.value = [...danmakuList.value.slice(-49), newDanmaku];
  }, 2000);
});

onBeforeUnmount(() => {
  if (intervalId) {
    window.clearInterval(intervalId);
  }
});

watch(
  danmakuList,
  () => {
    nextTick(() => {
      if (scrollRef.value) {
        scrollRef.value.scrollTop = scrollRef.value.scrollHeight;
      }
    });
  },
  { deep: true }
);

const handleSend = () => {
  if (!inputText.value.trim()) {
    return;
  }

  danmakuList.value = [
    ...danmakuList.value,
    {
      id: Date.now(),
      user: 'You',
      content: inputText.value,
      color: 'var(--accent)',
    },
  ];
  inputText.value = '';
};
</script>

<style scoped>
.player-shell {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 9;
  background-color: #000;
  border-radius: var(--radius-xl);
  overflow: hidden;
  border: 1px solid var(--border);
  box-shadow: var(--shadow-md);
}

.back-btn {
  position: absolute;
  top: 20px;
  left: 20px;
  z-index: 10;
  background-color: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(12px) saturate(180%);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: var(--radius-md);
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.2s ease, background-color 0.2s ease;
}

.back-btn:hover {
  transform: scale(1.1);
  background-color: rgba(0, 0, 0, 0.7);
}

.back-btn:active {
  transform: scale(0.9);
}

.player-cover {
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0.3;
}

.player-status {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  gap: 20px;
}

.player-pulse {
  width: 48px;
  height: 48px;
  border-radius: 14px;
  background-color: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  animation: pulse 2s ease-in-out infinite;
}

.player-dot {
  width: 8px;
  height: 8px;
  background-color: var(--accent);
  border-radius: 50%;
  box-shadow: 0 0 16px var(--accent);
}

.player-status-text {
  color: #fff;
  font-size: 0.8rem;
  font-weight: 700;
  opacity: 0.7;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}

.player-controls {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 24px 32px;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.8), transparent);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.volume-track {
  width: 120px;
  height: 4px;
  background-color: rgba(255, 255, 255, 0.2);
  border-radius: 2px;
}

.volume-level {
  width: 70%;
  height: 100%;
  background-color: #fff;
  border-radius: 2px;
}

.info-bar {
  padding: 24px;
  border-radius: var(--radius-xl);
  display: flex;
  align-items: center;
  justify-content: space-between;
  box-shadow: var(--shadow-low);
}

.info-avatar {
  width: 64px;
  height: 64px;
  border-radius: var(--radius-md);
  border: 1px solid var(--border);
}

.info-title {
  font-size: 1.25rem;
  font-weight: 800;
  color: var(--text-primary);
  margin-bottom: 6px;
  letter-spacing: -0.02em;
}

.info-meta {
  display: flex;
  align-items: center;
  gap: 14px;
}

.info-meta p {
  font-weight: 700;
  color: var(--text-primary);
  font-size: 0.95rem;
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
  font-size: 0.9rem;
  font-weight: 500;
}

.platform-tag {
  background-color: var(--accent-glow);
  color: var(--accent);
  padding: 4px 12px;
  border-radius: var(--radius-sm);
  font-size: 0.7rem;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 0.02em;
}

.follow-btn {
  background: linear-gradient(135deg, var(--accent), #0056b3);
  color: #fff;
  padding: 12px 32px;
  border-radius: var(--radius-lg);
  font-weight: 700;
  font-size: 0.95rem;
  box-shadow: 0 8px 20px var(--accent-glow);
  transition: transform 0.2s ease;
}

.follow-btn:hover {
  transform: translateY(-2px);
}

.share-btn {
  color: var(--text-secondary);
  padding: 12px;
  border-radius: var(--radius-lg);
  border: 1px solid var(--border);
  background-color: rgba(0, 0, 0, 0.02);
  transition: transform 0.2s ease, background-color 0.2s ease;
}

.share-btn:hover {
  transform: scale(1.05);
  background-color: rgba(0, 0, 0, 0.05);
}

.chat-panel {
  width: 380px;
  border-radius: var(--radius-xl);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: var(--shadow-low);
}

.chat-header {
  padding: 20px 24px;
  border-bottom: 1px solid var(--border);
  font-weight: 800;
  font-size: 1rem;
  color: var(--text-primary);
  letter-spacing: -0.02em;
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.chat-message {
  font-size: 0.9rem;
  line-height: 1.6;
}

.chat-user {
  font-weight: 800;
  margin-right: 10px;
}

.chat-text {
  color: var(--text-primary);
  font-weight: 500;
}

.chat-input-wrap {
  padding: 24px;
  border-top: 1px solid var(--border);
}

.chat-input-shell {
  display: flex;
  background-color: rgba(0, 0, 0, 0.04);
  padding: 14px 20px;
  border-radius: var(--radius-lg);
  gap: 14px;
  border: 1px solid transparent;
  box-shadow: var(--shadow-low);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

:global([data-theme='dark']) .chat-input-shell {
  background-color: rgba(255, 255, 255, 0.06);
}

.chat-input-shell.focused {
  transform: scale(1.02);
  box-shadow: var(--shadow-md);
}

.chat-input-shell input {
  flex: 1;
  border: none;
  background: none;
  outline: none;
  color: var(--text-primary);
  font-size: 0.9rem;
  font-weight: 500;
}

.chat-input-shell button {
  color: var(--accent);
  display: flex;
  align-items: center;
  transition: transform 0.2s ease;
}

.chat-input-shell button:hover {
  transform: scale(1.1);
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}
</style>
