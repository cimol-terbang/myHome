<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// ── Theme ──────────────────────────────────────────────────────────────────
const isDark = ref(false)

function initTheme() {
  const stored = localStorage.getItem('hp-theme')
  if (stored) {
    isDark.value = stored === 'dark'
  } else {
    isDark.value = false
  }
  applyTheme()
}

function applyTheme() {
  document.documentElement.setAttribute('data-theme', isDark.value ? 'dark' : 'light')
}

function toggleTheme() {
  isDark.value = !isDark.value
  localStorage.setItem('hp-theme', isDark.value ? 'dark' : 'light')
  applyTheme()
}

// ── Writing / Posts ────────────────────────────────────────────────────────
const posts = ref([])
const postsVisible = ref(false)

const NOTES_API = import.meta.env.VITE_NOTES_API_URL || 'https://note-api.haotian.my.id'
const NOTES_BASE = 'https://note.haotian.my.id'

function stripMarkdown(text) {
  if (!text) return ''
  return text
    // remove images: ![alt](url)
    .replace(/!\[.*?\]\(.*?\)/g, '')
    // remove links but keep text: [text](url)
    .replace(/\[([^\]]+)\]\([^)]+\)/g, '$1')
    // remove headings
    .replace(/^#{1,6}\s+/gm, '')
    // remove bold/italic
    .replace(/(\*{1,3}|_{1,3})(.*?)\1/g, '$2')
    // remove inline code
    .replace(/`[^`]+`/g, '')
    // remove blockquotes
    .replace(/^>\s+/gm, '')
    // remove horizontal rules
    .replace(/^[-*_]{3,}\s*$/gm, '')
    // collapse whitespace
    .replace(/\s+/g, ' ')
    .trim()
}

function truncate(text, len = 140) {
  const stripped = stripMarkdown(text)
  if (stripped.length <= len) return stripped
  return stripped.slice(0, len).trimEnd() + '…'
}

async function fetchPosts() {
  try {
    const { data } = await axios.get(`${NOTES_API}/posts`, { timeout: 6000 })
    const items = Array.isArray(data) ? data : (data.posts || data.data || [])
    posts.value = items.slice(0, 3)
    if (posts.value.length > 0) postsVisible.value = true
  } catch {
    // silently hide the section on error
    postsVisible.value = false
  }
}

function postUrl(post) {
  const slug = post.slug || post._id || post.id
  return `${NOTES_BASE}/post/${slug}`
}

function formatDate(dateStr) {
  if (!dateStr) return ''
  const d = new Date(dateStr)
  return d.toLocaleDateString('en-US', { year: 'numeric', month: 'short', day: 'numeric' })
}

// ── Nav scroll state ───────────────────────────────────────────────────────
const scrolled = ref(false)

function onScroll() {
  scrolled.value = window.scrollY > 40
}

// ── Lifecycle ──────────────────────────────────────────────────────────────
onMounted(() => {
  initTheme()
  fetchPosts()
  window.addEventListener('scroll', onScroll, { passive: true })
})
</script>

<template>
  <div class="site">

    <!-- ── Nav ──────────────────────────────────────────────────────────── -->
    <nav class="nav" :class="{ 'nav--scrolled': scrolled }">
      <div class="container nav__inner">
        <a href="#" class="nav__logo">haotian</a>
        <div class="nav__links">
          <a href="#about">about</a>
          <a href="#projects">projects</a>
          <a href="#writing" v-if="postsVisible">writing</a>
          <a href="#links">links</a>
        </div>
        <button class="theme-toggle" @click="toggleTheme" :aria-label="isDark ? 'Switch to light mode' : 'Switch to dark mode'">
          <i :class="isDark ? 'pi pi-sun' : 'pi pi-moon'"></i>
        </button>
      </div>
    </nav>

    <!-- ── Hero ─────────────────────────────────────────────────────────── -->
    <section class="hero" id="top">
      <div class="container hero__inner">
        <div class="hero__eyebrow">hey, I'm</div>
        <h1 class="hero__name">Haotian</h1>
        <p class="hero__tagline">
          building things on the backend, thinking things on the side.
        </p>
        <div class="hero__actions">
          <a href="#projects" class="btn btn--primary">see my work</a>
          <a href="#about" class="btn btn--ghost">about me</a>
        </div>
        <div class="hero__scroll-hint" aria-hidden="true">
          <i class="pi pi-angle-down"></i>
        </div>
      </div>
    </section>

    <!-- ── About ────────────────────────────────────────────────────────── -->
    <section class="section" id="about">
      <div class="container">
        <h2 class="section__title">about</h2>
        <div class="about__body">
          <p>
            I'm a backend-leaning developer who spends most of his time in JavaScript, APIs,
            databases, the kind of work that happens quietly behind the scenes. I like systems that are
            honest about what they do. Clean logic, clear structure.
          </p>
          <p>
            Outside of code, I sometimes spend time exploring philosophy and theology, mostly out of curiosity and personal interest. I'm drawn to questions that don't have simple answers, what we owe each other, how belief shapes the way we see the world, and why some ideas survive for centuries while others fade away. I also enjoy discussions around these topics, especially conversations that challenge perspectives or explore different ways of thinking. Gaming shows up sometimes too, usually when I need to turn my brain off for a few hours.
          </p>
          <p>
            This site is a small corner of the internet that's mine. The notes app is where I write things
            down essays, fragments, whatever's been sitting in my head. If something here resonates,
            feel free to reach out.
          </p>
        </div>
        <div class="about__tags">
          <span class="tag">JavaScript</span>
          <span class="tag">Node.js</span>
          <span class="tag">Sveltekit</span>
          <span class="tag">Vue 3</span>
          <span class="tag">philosophy</span>
          <span class="tag">reading</span>
          <span class="tag">backend</span>
          <span class="tag">Minecraft</span>
          <span class="tag">Mobile Legend</span>
        </div>
      </div>
    </section>

    <!-- ── Projects ─────────────────────────────────────────────────────── -->
    <section class="section section--alt" id="projects">
      <div class="container">
        <h2 class="section__title">projects</h2>
        <div class="projects__grid">

          <!-- my Notes -->
          <article class="project-card">
            <div class="project-card__header">
              <h3 class="project-card__name">my Notes</h3>
              <span class="badge badge--live">live</span>
            </div>
            <p class="project-card__desc">
              A personal writing space for essays, poems, and stories. Built to be quiet and readable —
              no distractions, just words.
            </p>
            <div class="project-card__stack">
              <span class="stack-badge">Vue 3</span>
              <span class="stack-badge">Node.js</span>
              <span class="stack-badge">MongoDB</span>
            </div>
            <a
              href="https://note.haotian.my.id"
              target="_blank"
              rel="noopener noreferrer"
              class="project-card__link"
            >
              visit <i class="pi pi-arrow-up-right"></i>
            </a>
          </article>

          <!-- ask.haotian.my.id -->
          <article class="project-card project-card--muted">
            <div class="project-card__header">
              <h3 class="project-card__name">ask.haotian</h3>
              <span class="badge badge--dev">in development</span>
            </div>
            <p class="project-card__desc">
              An anonymous Q&amp;A platform — a place to ask things you might not ask out loud.
            </p>
            <div class="project-card__stack">
              <span class="stack-badge">Vue 3</span>
              <span class="stack-badge">Node.js</span>
            </div>
          </article>



    <!-- mc -->
          <article class="project-card project-card--muted">
            <div class="project-card__header">
              <h3 class="project-card__name">Minecraft addon</h3>
              <span class="badge badge--dev">in development</span>
            </div>
            <p class="project-card__desc">
              Just a custom Minecraft behavior pack.
            </p>
            <div class="project-card__stack">
              <span class="stack-badge">bedrock</span>
              <span class="stack-badge">javascript</span>
            </div>
          </article>

        </div>
      </div>
    </section>

    <!-- ── Writing ───────────────────────────────────────────────────────── -->
    <section class="section" id="writing" v-if="postsVisible">
      <div class="container">
        <div class="section__header">
          <h2 class="section__title">writing</h2>
          <a href="https://note.haotian.my.id" target="_blank" rel="noopener noreferrer" class="section__more">
            all posts <i class="pi pi-arrow-right"></i>
          </a>
        </div>
        <div class="writing__grid">
          <a
            v-for="post in posts"
            :key="post._id || post.id || post.slug"
            :href="postUrl(post)"
            target="_blank"
            rel="noopener noreferrer"
            class="writing-card"
          >
            <div class="writing-card__meta">
              <span v-if="post.category" class="writing-card__type" :class="`writing-card__type--${post.category}`">
                {{ post.category }}
              </span>
              <span class="writing-card__date">{{ formatDate(post.createdAt || post.date) }}</span>
            </div>
            <h3 class="writing-card__title">{{ post.title }}</h3>
            <p class="writing-card__preview">{{ truncate(post.content || post.body || post.excerpt) }}</p>
          </a>
        </div>
      </div>
    </section>

    <!-- ── Links ─────────────────────────────────────────────────────────── -->
    <section class="section section--alt" id="links">
      <div class="container">
        <h2 class="section__title">links</h2>
        <div class="links__grid">

          <a
            href="https://github.com/cimol-terbang"
            target="_blank"
            rel="noopener noreferrer"
            class="link-card"
          >
            <i class="pi pi-github link-card__icon"></i>
            <div>
              <div class="link-card__label">GitHub</div>
              <div class="link-card__value">cimol-terbang</div>
            </div>
          </a>

          <!-- TODO: update with real link -->
          <a
            href="https://instagram.com/cimol.chwan"
            target="_blank"
            rel="noopener noreferrer"
            class="link-card"
            aria-label="Instagram"
          >
            <i class="pi pi-instagram link-card__icon"></i>
            <div>
              <div class="link-card__label">Instagram</div>
              <div class="link-card__value">@cimol.chwan</div>
            </div>
          </a>

          <!-- Reality -->
          <a
            href="https://reality.app/profile/ee93e8e1?adj_t=8ogcewh_z9yhix5"
            target="_blank"
            rel="noopener noreferrer"
            class="link-card"
            aria-label="Reality"
          >
            <span class="reality-icon" aria-hidden="true">
              <svg width="22" height="22" viewBox="0 0 100 100" fill="none" xmlns="http://www.w3.org/2000/svg">
                <defs>
                  <linearGradient id="rg" x1="0%" y1="0%" x2="100%" y2="100%">
                    <stop offset="0%" stop-color="#60b8ff"/>
                    <stop offset="45%" stop-color="#9b6bff"/>
                    <stop offset="100%" stop-color="#ff2dcc"/>
                  </linearGradient>
                </defs>
                <rect x="18" y="18" width="64" height="64" rx="18" ry="18"
                  transform="rotate(45 50 50)"
                  fill="url(#rg)"
                />
                <rect x="28" y="28" width="44" height="44" rx="12" ry="12"
                  transform="rotate(45 50 50)"
                  fill="white"
                />
              </svg>
            </span>
            <div>
              <div class="link-card__label">Reality</div>
              <div class="link-card__value">uɐı̣ʇoɐH</div>
            </div>
          </a>

          <div class="link-card" aria-label="Discord">
            <i class="pi pi-discord link-card__icon"></i>
            <div>
              <div class="link-card__label">Discord</div>
              <div class="link-card__value">__haotian</div>
            </div>
          </div>

        </div>

        <p class="links__note">
          want to ask something anonymously?
          <span class="links__note-accent">ask.haotian.my.id</span>
          is on its way.
        </p>
      </div>
    </section>

    <!-- ── Footer ────────────────────────────────────────────────────────── -->
    <footer class="footer">
      <div class="container footer__inner">
        <span>haotian.my.id</span>
        <span class="footer__sep">·</span>
        <span>built with Vue 3</span>
      </div>
    </footer>

  </div>
</template>

<style scoped>
/* ── Site wrapper ──────────────────────────────────────── */
.site {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* ── Nav ───────────────────────────────────────────────── */
.nav {
  position: fixed;
  top: 0; left: 0; right: 0;
  z-index: 100;
  padding: 1rem 0;
  transition: background var(--mn-transition), box-shadow var(--mn-transition), padding var(--mn-transition);
}

.nav--scrolled {
  background: var(--nav-scrolled-bg);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  box-shadow: 0 1px 0 var(--mn-border), 0 2px 16px rgba(59,130,196,0.08);
  padding: 0.6rem 0;
}

.nav__inner {
  display: flex;
  align-items: center;
  gap: 2rem;
}

.nav__logo {
  font-weight: 800;
  font-size: 1.05rem;
  color: var(--mn-text);
  letter-spacing: -0.03em;
  margin-right: auto;
}

.nav__logo::after {
  content: '.';
  color: var(--mn-accent);
}

.nav__links {
  display: flex;
  gap: 1.5rem;
}

.nav__links a {
  font-size: 0.875rem;
  color: var(--mn-text-muted);
  font-weight: 500;
  transition: color var(--mn-transition);
}

.nav__links a:hover { color: var(--mn-text); }

.theme-toggle {
  background: var(--mn-surface);
  border: 1px solid var(--mn-border);
  border-radius: 20px;
  color: var(--mn-text-muted);
  padding: 0.3rem 0.6rem;
  font-size: 0.85rem;
  transition: all var(--mn-transition);
  display: flex;
  align-items: center;
  box-shadow: var(--mn-shadow-sm);
}

.theme-toggle:hover {
  color: var(--mn-text);
  border-color: var(--mn-accent);
}

/* ── Hero ──────────────────────────────────────────────── */
.hero {
  min-height: 100vh;
  display: flex;
  align-items: center;
  padding: 6rem 0 0;
  position: relative;
  overflow: hidden;
}

.hero::before {
  content: '';
  position: absolute;
  top: 15%; right: -10%;
  width: 420px; height: 420px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(59,130,196,0.13) 0%, transparent 70%);
  pointer-events: none;
}

.hero::after {
  content: '';
  position: absolute;
  bottom: 10%; left: -8%;
  width: 300px; height: 300px;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(96,165,216,0.10) 0%, transparent 70%);
  pointer-events: none;
}

.hero__inner {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  position: relative;
  z-index: 1;
}

.hero__eyebrow {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.82rem;
  color: var(--mn-accent);
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.hero__eyebrow::before {
  content: '';
  display: inline-block;
  width: 22px; height: 2px;
  background: var(--mn-accent);
  border-radius: 2px;
}

.hero__name {
  font-size: clamp(3rem, 8vw, 5.5rem);
  font-weight: 800;
  letter-spacing: -0.04em;
  line-height: 1.05;
  background: linear-gradient(135deg, var(--mn-text) 30%, var(--mn-accent) 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero__tagline {
  font-size: clamp(1rem, 2.5vw, 1.15rem);
  color: var(--mn-text-muted);
  max-width: 460px;
  line-height: 1.65;
  margin-top: 0.25rem;
}

.hero__actions {
  display: flex;
  gap: 0.75rem;
  margin-top: 0.75rem;
  flex-wrap: wrap;
}

.hero__scroll-hint {
  margin-top: 2rem;
  color: var(--mn-text-subtle);
  font-size: 1.1rem;
  animation: bounce 2.2s ease-in-out infinite;
  width: fit-content;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(7px); }
}

/* ── Buttons ───────────────────────────────────────────── */
.btn {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.6rem 1.3rem;
  border-radius: 24px;
  font-size: 0.875rem;
  font-weight: 600;
  transition: all var(--mn-transition);
  border: 1.5px solid transparent;
  letter-spacing: -0.01em;
}

.btn--primary {
  background: var(--mn-accent);
  color: #fff;
  box-shadow: 0 4px 14px rgba(59,130,196,0.35);
}

.btn--primary:hover {
  background: var(--mn-accent-hover);
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(59,130,196,0.45);
}

.btn--ghost {
  border-color: var(--mn-border);
  color: var(--mn-text-muted);
  background: var(--mn-surface);
  box-shadow: var(--mn-shadow-sm);
}

.btn--ghost:hover {
  border-color: var(--mn-accent);
  color: var(--mn-accent);
  transform: translateY(-1px);
  box-shadow: var(--mn-shadow);
}

/* ── Sections ──────────────────────────────────────────── */
.section {
  padding: 5rem 0;
  position: relative;
}

.section--alt {
  background: var(--mn-surface);
  border-top: 1px solid var(--mn-border);
  border-bottom: 1px solid var(--mn-border);
}

.section__title {
  font-size: 0.72rem;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--mn-accent);
  margin-bottom: 2rem;
  display: flex;
  align-items: center;
  gap: 0.6rem;
}

.section__title::after {
  content: '';
  flex: 1;
  max-width: 36px;
  height: 1.5px;
  background: var(--mn-border);
  border-radius: 2px;
}

.section__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 2rem;
}

.section__header .section__title { margin-bottom: 0; }

.section__more {
  font-size: 0.82rem;
  color: var(--mn-accent);
  display: flex;
  align-items: center;
  gap: 0.3rem;
  font-weight: 600;
  transition: color var(--mn-transition);
}

.section__more:hover { color: var(--mn-accent-hover); }

/* ── About ─────────────────────────────────────────────── */
.about__body {
  display: flex;
  flex-direction: column;
  gap: 1.1rem;
  color: var(--mn-text-muted);
  font-size: 1rem;
  line-height: 1.8;
  max-width: 620px;
  margin-bottom: 2rem;
}

.about__tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.tag {
  background: var(--mn-surface);
  border: 1px solid var(--mn-border);
  color: var(--mn-text-muted);
  font-size: 0.78rem;
  font-weight: 500;
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  box-shadow: var(--mn-shadow-sm);
  transition: all var(--mn-transition);
}

.tag:hover {
  border-color: var(--mn-accent);
  color: var(--mn-accent);
}

/* ── Projects ──────────────────────────────────────────── */
.projects__grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 1.25rem;
}

.project-card {
  background: var(--mn-bg);
  border: 1px solid var(--mn-border);
  border-radius: var(--mn-radius);
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  transition: all var(--mn-transition);
  position: relative;
  overflow: hidden;
}

.project-card::before {
  content: '';
  position: absolute;
  top: 0; left: 1.5rem; right: 1.5rem;
  height: 2px;
  background: linear-gradient(90deg, var(--mn-accent), transparent);
  border-radius: 0 0 2px 2px;
  opacity: 0;
  transition: opacity var(--mn-transition);
}

.project-card:hover {
  box-shadow: var(--mn-shadow);
  transform: translateY(-3px);
  border-color: var(--mn-accent);
}

.project-card:hover::before { opacity: 1; }
.project-card--muted { opacity: 0.7; }
.project-card--muted:hover { opacity: 1; }

.project-card__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.5rem;
}

.project-card__name {
  font-size: 1rem;
  font-weight: 700;
  color: var(--mn-text);
  letter-spacing: -0.01em;
}

.project-card__desc {
  font-size: 0.875rem;
  color: var(--mn-text-muted);
  line-height: 1.65;
  flex: 1;
}

.project-card__stack {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
}

.stack-badge {
  background: var(--mn-surface-2);
  border: 1px solid var(--mn-border);
  color: var(--mn-text-subtle);
  font-size: 0.72rem;
  font-weight: 500;
  padding: 0.15rem 0.55rem;
  border-radius: 20px;
}

.project-card__link {
  display: inline-flex;
  align-items: center;
  gap: 0.3rem;
  font-size: 0.82rem;
  color: var(--mn-accent);
  font-weight: 600;
  margin-top: auto;
  transition: color var(--mn-transition);
}

.project-card__link:hover { color: var(--mn-accent-hover); }

/* ── Badges ────────────────────────────────────────────── */
.badge {
  font-size: 0.67rem;
  font-weight: 700;
  padding: 0.18rem 0.55rem;
  border-radius: 20px;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  white-space: nowrap;
}

.badge--live {
  background: rgba(34,197,94,0.12);
  color: #16a34a;
  border: 1px solid rgba(34,197,94,0.25);
}

[data-theme='dark'] .badge--live {
  background: rgba(34,197,94,0.1);
  color: #4ade80;
  border-color: rgba(34,197,94,0.2);
}

.badge--soon {
  background: var(--mn-surface-2);
  color: var(--mn-text-subtle);
  border: 1px solid var(--mn-border);
}

.badge--dev {
  background: rgba(124, 106, 247, 0.1);
  color: var(--mn-accent);
  border: 1px solid rgba(124, 106, 247, 0.2);
}

/* ── Writing ───────────────────────────────────────────── */
.writing__grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 1.25rem;
}

.writing-card {
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
  background: var(--mn-bg);
  border: 1px solid var(--mn-border);
  border-radius: var(--mn-radius);
  padding: 1.25rem;
  transition: all var(--mn-transition);
  color: var(--mn-text);
  position: relative;
}

.writing-card--essay { border-top: 3px solid #3b82f6; }
.writing-card--poem  { border-top: 3px solid #ec4899; }
.writing-card--story { border-top: 3px solid #22c55e; }

.writing-card:hover {
  box-shadow: var(--mn-shadow);
  transform: translateY(-3px);
}

.writing-card__meta {
  display: flex;
  align-items: center;
  gap: 0.6rem;
}

.writing-card__type {
  font-size: 0.67rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  padding: 0.15rem 0.5rem;
  border-radius: 20px;
}

.writing-card__type--essay { background: #e0eeff; color: #1d4ed8; }
.writing-card__type--poem  { background: #fce7f3; color: #be185d; }
.writing-card__type--story { background: #dcfce7; color: #15803d; }

.writing-card__date {
  font-size: 0.75rem;
  color: var(--mn-text-subtle);
}

.writing-card__title {
  font-size: 0.95rem;
  font-weight: 700;
  color: var(--mn-text);
  line-height: 1.4;
  letter-spacing: -0.01em;
}

.writing-card__preview {
  font-size: 0.845rem;
  color: var(--mn-text-muted);
  line-height: 1.65;
  flex: 1;
}

/* ── Links ─────────────────────────────────────────────── */
.links__grid {
  display: flex;
  flex-wrap: wrap;
  gap: 0.875rem;
  margin-bottom: 2rem;
}

.link-card {
  display: flex;
  align-items: center;
  gap: 0.875rem;
  background: var(--mn-surface);
  border: 1px solid var(--mn-border);
  border-radius: var(--mn-radius);
  padding: 0.875rem 1.25rem;
  min-width: 170px;
  transition: all var(--mn-transition);
  color: var(--mn-text);
  box-shadow: var(--mn-shadow-sm);
}

.link-card:not(.link-card--placeholder):hover {
  box-shadow: var(--mn-shadow);
  transform: translateY(-2px);
  border-color: var(--mn-accent);
}

.link-card--placeholder { opacity: 0.45; cursor: default; }

.link-card__icon {
  font-size: 1.3rem;
  color: var(--mn-accent);
  opacity: 0.8;
}

.reality-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 1.3rem;
  height: 1.3rem;
  flex-shrink: 0;
}

.link-card__label {
  font-size: 0.7rem;
  color: var(--mn-text-subtle);
  text-transform: uppercase;
  letter-spacing: 0.06em;
  font-weight: 600;
}

.link-card__value {
  font-size: 0.9rem;
  font-weight: 600;
  color: var(--mn-text);
}

.links__note {
  font-size: 0.875rem;
  color: var(--mn-text-subtle);
}

.links__note-accent {
  color: var(--mn-accent);
  font-weight: 600;
}

/* ── Footer ────────────────────────────────────────────── */
.footer {
  margin-top: auto;
  padding: 1.75rem 0;
  border-top: 1px solid var(--mn-border);
  background: var(--mn-surface);
}

.footer__inner {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.82rem;
  color: var(--mn-text-subtle);
}

.footer__sep { opacity: 0.4; }

/* ── Responsive ────────────────────────────────────────── */
@media (max-width: 640px) {
  .nav__links { display: none; }
  .hero { padding: 5rem 0 3rem; }
  .hero::before, .hero::after { display: none; }
  .projects__grid, .writing__grid { grid-template-columns: 1fr; }
  .links__grid { flex-direction: column; }
  .link-card { min-width: unset; }
}
</style>
