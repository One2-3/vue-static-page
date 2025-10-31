<template>
  <main class="scene" role="main" aria-label="한가위 테마 페이지">
    <!-- 배경 하늘, 별, 달 -->
    <div class="sky" aria-hidden="true">
      <div class="stars"></div>
      <div class="moon">
        <!-- 달토끼 실루엣 (inline SVG) -->
        <svg class="rabbit" viewBox="0 0 120 120">
          <g fill="currentColor" opacity="0.28">
            <path
              d="M64 24c-5 0-9 4-9 9 0 2 1 4 2 6-8-1-16 4-19 12-3 7-1 16 5 21-7 3-12 9-12 16 0 10 11 18 25 18s25-8 25-18c0-7-5-13-12-16 6-5 8-14 5-21-3-8-11-13-19-12 2-2 2-4 2-6 0-5-4-9-9-9z"
            />
            <circle cx="78" cy="38" r="6" />
            <path
              d="M84 48c6 1 12 6 14 12 2 6-1 13-6 16-2 1-4 2-6 2 2-3 3-6 3-9 0-8-4-15-11-21 2 0 4 0 6 0z"
            />
          </g>
        </svg>
      </div>

      <!-- 연등 (lanterns) -->
      <div
        v-if="showLanterns"
        v-for="lantern in lanterns"
        :key="lantern.id"
        class="lantern"
        :style="lanternStyle(lantern)"
        aria-hidden="true"
      >
        <div class="string"></div>
        <div class="body"></div>
        <div class="glow"></div>
        <div class="tail"></div>
      </div>
    </div>

    <!-- 콘텐츠 -->
    <section class="hero">
      <h1 class="title">풍요로운 한가위</h1>
      <p class="subtitle">
        보름달처럼 마음도 가득 채우는 명절 되세요<span class="sparkle"> ✨</span>
      </p>

      <div class="cta">
        <button class="btn" @click="openWish" aria-controls="wish-modal">소원 빌기</button>
        <button class="btn ghost" @click="toggleLanterns">
          연등 {{ showLanterns ? '끄기' : '켜기' }}
        </button>
      </div>

      <small class="foot">Made with Vue · TypeScript · Vite</small>
    </section>

    <!-- 소원 모달 -->
    <div
      v-if="showModal"
      id="wish-modal"
      class="modal"
      role="dialog"
      aria-modal="true"
      aria-label="달님에게 소원 빌기"
      @click.self="closeWish"
    >
      <div class="card">
        <h2>달님에게 소원 빌기 🌕</h2>
        <form @submit.prevent="saveWish">
          <textarea
            v-model="wish"
            :maxlength="120"
            placeholder="건강, 행복, 풍성함… 한 줄로 적어보세요 (최대 120자)"
            aria-label="소원 입력"
            autofocus
          />
          <div class="actions">
            <button class="btn" type="submit">전하기</button>
            <button class="btn ghost" type="button" @click="closeWish">닫기</button>
          </div>
        </form>
        <p v-if="savedMessage" class="saved">
          {{ savedMessage }}
          <button class="link" type="button" @click="copyWish">복사</button>
        </p>
      </div>
    </div>
  </main>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

type Lantern = {
  id: number
  x: number // 0~100 vw 위치
  scale: number // 크기
  duration: number // 부유 시간
  delay: number // 시작 지연
}

const showLanterns = ref(true)
const showModal = ref(false)
const wish = ref('')
const savedMessage = ref('')

// 초기 연등 배치
const lanterns = ref<Lantern[]>(
  Array.from({ length: 10 }, (_, i) => ({
    id: i,
    x: 5 + Math.random() * 90,
    scale: 0.6 + Math.random() * 0.9,
    duration: 18 + Math.random() * 16,
    delay: Math.random() * 10
  }))
)

function lanternStyle(l: Lantern) {
  return {
    left: `${l.x}%`,
    animationDuration: `${l.duration}s`,
    animationDelay: `${l.delay}s`,
    transform: `translateY(0) scale(${l.scale})`
  } as Record<string, string>
}

function toggleLanterns() {
  showLanterns.value = !showLanterns.value
}

function openWish() {
  // 저장된 소원 불러오기
  const saved = localStorage.getItem('chuseok_wish')
  wish.value = saved || ''
  savedMessage.value = ''
  showModal.value = true
}

function closeWish() {
  showModal.value = false
}

function saveWish() {
  localStorage.setItem('chuseok_wish', wish.value.trim())
  savedMessage.value = '달에게 전해졌어요! 좋은 일만 가득하길 💫'
}

async function copyWish() {
  try {
    await navigator.clipboard.writeText(wish.value)
    savedMessage.value = '클립보드에 복사했습니다 📋'
  } catch {
    savedMessage.value = '복사에 실패했어요. 직접 선택해 복사해 주세요.'
  }
}

const greeting = computed(() => {
  const h = new Date().getHours()
  if (h < 6) return '고요한 새벽'
  if (h < 12) return '상쾌한 아침'
  if (h < 18) return '따스한 오후'
  return '은은한 밤'
})
</script>

<style scoped>
/* 색상 변수 */
:root {
  --bg-1: #0b1022;
  --bg-2: #1a2140;
  --accent: #ffd166;
  --accent-2: #ffedb6;
  --ink: #e5e7eb;
  --muted: #aab1c5;
  --gold: #ffcc66;
  --emerald: #30e0a1;
}

/* 전체 장면 */
.scene {
  min-height: 100vh;
  color: var(--ink);
  background: radial-gradient(1200px 800px at 70% -10%, #223 0%, transparent 60%),
    linear-gradient(180deg, var(--bg-1), var(--bg-2));
  position: relative;
  overflow: hidden;
  font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, 'Apple SD Gothic Neo',
    'Noto Sans KR', 'Malgun Gothic', 'Segoe UI Symbol', sans-serif;
}

/* 하늘 층 */
.sky {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

/* 별 */
.stars {
  position: absolute;
  inset: -30% -30% -20% -30%;
  background-image:
    radial-gradient(1px 1px at 10% 20%, rgba(255,255,255,0.9), transparent 60%),
    radial-gradient(1.2px 1.2px at 30% 80%, rgba(255,255,255,0.7), transparent 60%),
    radial-gradient(1.5px 1.5px at 50% 40%, rgba(255,255,255,0.8), transparent 60%),
    radial-gradient(1px 1px at 70% 30%, rgba(255,255,255,0.7), transparent 60%),
    radial-gradient(1.2px 1.2px at 85% 75%, rgba(255,255,255,0.8), transparent 60%),
    radial-gradient(1px 1px at 20% 60%, rgba(255,255,255,0.9), transparent 60%);
  animation: drift 60s linear infinite;
  opacity: 0.9;
}

@keyframes drift {
  from { transform: translateY(0) rotate(0deg); }
  to { transform: translateY(-6%) rotate(0.4deg); }
}

/* 달 */
.moon {
  position: absolute;
  top: clamp(24px, 6vh, 120px);
  right: clamp(24px, 6vw, 120px);
  width: clamp(140px, 17vw, 260px);
  aspect-ratio: 1 / 1;
  border-radius: 50%;
  background: radial-gradient(100px 100px at 30% 30%, #fffdf0 0%, #fff1a8 40%, #ffd166 62%, #fdb949 78%, #f7a53c 100%);
  box-shadow: 0 0 60px 10px rgba(255, 213, 94, 0.35), 0 0 140px 30px rgba(255, 237, 182, 0.15);
  filter: saturate(1.05);
}
.moon .rabbit {
  position: absolute;
  inset: 0;
  color: #5d4a1f;
}

/* 연등 */
.lantern {
  --rise: -110vh;
  position: absolute;
  bottom: -18vh;
  transform-origin: 50% 100%;
  animation: float var(--dur, 24s) linear infinite;
  will-change: transform, opacity;
  opacity: 0.85;
}

@keyframes float {
  0% { transform: translateY(0) scale(var(--scale, 1)) translateX(0); opacity: 0; }
  5% { opacity: 0.9; }
  50% { transform: translateY(calc(var(--rise) * 0.5)) scale(var(--scale, 1)) translateX(8px); }
  100% { transform: translateY(var(--rise)) scale(var(--scale, 1)) translateX(-8px); opacity: 0.6; }
}

.lantern .string {
  width: 2px;
  height: 18px;
  margin: 0 auto;
  background: linear-gradient(#b38b00, #886700);
  border-radius: 2px;
}
.lantern .body {
  width: 36px;
  height: 42px;
  margin: 0 auto;
  border-radius: 10px 10px 12px 12px / 12px 12px 14px 14px;
  background: linear-gradient(180deg, #ffefb0, #ffcc66 55%, #f1a83b);
  box-shadow: inset 0 0 0 2px rgba(90, 50, 0, 0.25);
  position: relative;
}
.lantern .body::before,
.lantern .body::after {
  content: '';
  position: absolute;
  left: 4px; right: 4px;
  height: 2px;
  background: rgba(120, 60, 0, 0.25);
  top: 10px;
}
.lantern .body::after { top: 22px; }
.lantern .glow {
  position: absolute;
  width: 70px;
  height: 70px;
  left: 50%;
  top: 28px;
  transform: translateX(-50%);
  border-radius: 50%;
  background: radial-gradient(closest-side, rgba(255, 215, 120, 0.55), transparent 70%);
  filter: blur(1px);
  pointer-events: none;
}
.lantern .tail {
  width: 12px;
  height: 8px;
  margin: 2px auto 0;
  background: linear-gradient(180deg, #7a4a00, #5c3800);
  border-radius: 0 0 6px 6px;
}

/* 히어로 텍스트 */
.hero {
  position: relative;
  z-index: 1;
  text-align: center;
  padding: min(14vh, 12rem) 1.25rem 8vh;
  display: grid;
  gap: 1rem;
  place-items: center;
}

.title {
  font-size: clamp(2rem, 5vw, 4rem);
  line-height: 1.05;
  letter-spacing: 0.02em;
  font-weight: 800;
  text-shadow: 0 2px 0 rgba(0,0,0,0.15), 0 0 24px rgba(255, 225, 120, 0.16);
}

.subtitle {
  color: var(--muted);
  font-size: clamp(0.95rem, 2.4vw, 1.2rem);
}
.sparkle { filter: drop-shadow(0 0 6px rgba(255, 215, 102, 0.4)); }

.cta {
  display: flex;
  gap: 0.8rem;
  margin-top: 0.75rem;
  flex-wrap: wrap;
  justify-content: center;
}

.btn {
  appearance: none;
  border: none;
  padding: 0.75rem 1.1rem;
  border-radius: 14px;
  background: linear-gradient(180deg, #ffd166, #f7b733);
  color: #2a1800;
  font-weight: 700;
  letter-spacing: 0.01em;
  cursor: pointer;
  box-shadow: 0 10px 24px rgba(255, 209, 102, 0.22), inset 0 0 0 1px rgba(110, 70, 0, 0.2);
  transition: transform 0.12s ease, box-shadow 0.2s ease;
}
.btn:hover { transform: translateY(-1px); }
.btn:active { transform: translateY(0); }

.btn.ghost {
  background: rgba(255, 255, 255, 0.06);
  color: var(--ink);
  box-shadow: inset 0 0 0 1px rgba(255, 255, 255, 0.18);
}

.foot {
  margin-top: 1rem;
  color: rgba(255, 255, 255, 0.45);
}

/* 모달 */
.modal {
  position: fixed;
  inset: 0;
  background: rgba(5, 8, 20, 0.6);
  backdrop-filter: blur(6px);
  display: grid;
  place-items: center;
  z-index: 5;
}
.card {
  width: min(92vw, 560px);
  background: linear-gradient(180deg, #161b2e, #11162a);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 18px;
  box-shadow: 0 30px 80px rgba(0, 0, 0, 0.45), 0 0 0 1px rgba(255, 220, 100, 0.06) inset;
  padding: 1.25rem;
}
.card h2 {
  margin: 0 0 0.5rem;
  font-size: 1.25rem;
}
.card textarea {
  width: 100%;
  min-height: 120px;
  resize: vertical;
  padding: 0.9rem 1rem;
  border-radius: 12px;
  background: #0f1430;
  color: var(--ink);
  border: 1px solid rgba(255, 255, 255, 0.12);
  outline: none;
  box-shadow: 0 0 0 2px transparent;
  transition: box-shadow 0.15s ease, border-color 0.15s ease;
}
.card textarea:focus {
  border-color: rgba(255, 209, 102, 0.55);
  box-shadow: 0 0 0 3px rgba(255, 209, 102, 0.2);
}
.actions {
  display: flex;
  gap: 0.6rem;
  margin-top: 0.75rem;
  justify-content: flex-end;
}
.saved {
  margin-top: 0.75rem;
  color: var(--accent-2);
}
.link {
  background: none;
  border: none;
  color: var(--emerald);
  font-weight: 700;
  cursor: pointer;
  margin-left: 0.25rem;
}

/* 반응형 미세 조정 */
@media (max-width: 520px) {
  .moon { right: 16px; top: 16px; }
  .btn { padding: 0.7rem 1rem; }
  .lantern .body { width: 30px; height: 36px; }
}

/* 유틸: v-for 스타일 값 전달용 */
.lantern {
  --dur: 24s;
  --scale: 1;
}
</style>
