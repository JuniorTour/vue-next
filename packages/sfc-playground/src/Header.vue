<script setup lang="ts">
import { downloadProject } from './download/download'
import { ref, onMounted } from 'vue'

// @ts-ignore
const { store } = defineProps(['store'])

const currentCommit = __COMMIT__
const activeVersion = ref(`@${currentCommit}`)
const publishedVersions = ref<string[]>()
const expanded = ref(false)

async function toggle() {
  expanded.value = !expanded.value
  if (!publishedVersions.value) {
    publishedVersions.value = await fetchVersions()
  }
}

async function setVueVersion(v: string) {
  activeVersion.value = `loading...`
  await store.setVueVersion(v)
  activeVersion.value = `v${v}`
  expanded.value = false
}

function resetVueVersion() {
  store.resetVueVersion()
  activeVersion.value = `@${currentCommit}`
  expanded.value = false
}

async function copyLink() {
  await navigator.clipboard.writeText(location.href)
  alert('Sharable URL has been copied to clipboard.')
}

onMounted(async () => {
  window.addEventListener('click', () => {
    expanded.value = false
  })
})

async function fetchVersions(): Promise<string[]> {
  const res = await fetch(
    `https://api.github.com/repos/vuejs/vue-next/releases?per_page=100`
  )
  const releases: any[] = await res.json()
  const versions = releases.map(r =>
    /^v/.test(r.tag_name) ? r.tag_name.substr(1) : r.tag_name
  )
  // if the latest version is a pre-release, list all current pre-releases
  // otherwise filter out pre-releases
  let isInPreRelease = versions[0].includes('-')
  const filteredVersions: string[] = []
  for (const v of versions) {
    if (v.includes('-')) {
      if (isInPreRelease) {
        filteredVersions.push(v)
      }
    } else {
      filteredVersions.push(v)
      isInPreRelease = false
    }
    if (filteredVersions.length >= 30 || v === '3.0.10') {
      break
    }
  }
  return filteredVersions
}
</script>

<template>
  <nav>
    <h1>
      <img alt="logo" src="/logo.svg" />
      <span>Vue SFC Playground</span>
    </h1>
    <div class="links">
      <div class="version" @click.stop>
        <span class="active-version" @click="toggle">
          Version: {{ activeVersion }}
        </span>
        <ul class="versions" :class="{ expanded }">
          <li v-if="!publishedVersions"><a>loading versions...</a></li>
          <li v-for="version of publishedVersions">
            <a @click="setVueVersion(version)">v{{ version }}</a>
          </li>
          <li>
            <a @click="resetVueVersion">This Commit ({{ currentCommit }})</a>
          </li>
          <li>
            <a
              href="https://app.netlify.com/sites/vue-sfc-playground/deploys"
              target="_blank"
              >Commits History</a
            >
          </li>
        </ul>
      </div>
      <button class="share" @click="copyLink">
        <svg width="1.4em" height="1.4em" viewBox="0 0 24 24">
          <g
            fill="none"
            stroke="#666"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <circle cx="18" cy="5" r="3" />
            <circle cx="6" cy="12" r="3" />
            <circle cx="18" cy="19" r="3" />
            <path d="M8.59 13.51l6.83 3.98" />
            <path d="M15.41 6.51l-6.82 3.98" />
          </g>
        </svg>
      </button>
      <button class="download" @click="downloadProject(store)">
        <svg width="1.7em" height="1.7em" viewBox="0 0 24 24">
          <g fill="#666">
            <rect x="4" y="18" width="16" height="2" rx="1" ry="1" />
            <rect
              x="3"
              y="17"
              width="4"
              height="2"
              rx="1"
              ry="1"
              transform="rotate(-90 5 18)"
            />
            <rect
              x="17"
              y="17"
              width="4"
              height="2"
              rx="1"
              ry="1"
              transform="rotate(-90 19 18)"
            />
            <path
              d="M12 15a1 1 0 0 1-.58-.18l-4-2.82a1 1 0 0 1-.24-1.39a1 1 0 0 1 1.4-.24L12 12.76l3.4-2.56a1 1 0 0 1 1.2 1.6l-4 3a1 1 0 0 1-.6.2z"
            />
            <path d="M12 13a1 1 0 0 1-1-1V4a1 1 0 0 1 2 0v8a1 1 0 0 1-1 1z" />
          </g>
        </svg>
      </button>
    </div>
  </nav>
</template>

<style>
nav {
  --bg: #fff;
  --bg-light: #fff;
  --border: #ddd;

  color: var(--base);
  height: var(--nav-height);
  box-sizing: border-box;
  padding: 0 1em;
  background-color: var(--bg);
  box-shadow: 0 0 4px rgba(0, 0, 0, 0.33);
  position: relative;
  z-index: 999;
  display: flex;
  justify-content: space-between;
}

.dark nav {
  --base: #ddd;
  --bg: #1a1a1a;
  --bg-light: #242424;
  --border: #383838;

  box-shadow: none;
  border-bottom: 1px solid var(--border);
}

h1 {
  margin: 0;
  line-height: var(--nav-height);
  font-weight: 500;
  display: inline-block;
  vertical-align: middle;
}

h1 img {
  height: 24px;
  vertical-align: middle;
  margin-right: 10px;
  position: relative;
  top: -2px;
}

@media (max-width: 480px) {
  h1 span {
    display: none;
  }
}

.links {
  display: flex;
}

.version {
  display: inline-block;
  margin-right: 12px;
  position: relative;
}

.active-version {
  cursor: pointer;
  position: relative;
  display: inline-block;
  vertical-align: middle;
  line-height: var(--nav-height);
  padding-right: 15px;
}

.active-version:after {
  content: '';
  width: 0;
  height: 0;
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-top: 6px solid #aaa;
  position: absolute;
  right: 0;
  top: 22px;
}

.version:hover .active-version:after {
  border-top-color: var(--base);
}

.dark .version:hover .active-version:after {
  border-top-color: #fff;
}

.versions {
  display: none;
  position: absolute;
  left: 0;
  top: 40px;
  background-color: var(--bg-light);
  border: 1px solid var(--border);
  border-radius: 4px;
  list-style-type: none;
  padding: 8px;
  margin: 0;
  width: 200px;
  max-height: calc(100vh - 70px);
  overflow: scroll;
}

.versions a {
  display: block;
  padding: 6px 12px;
  text-decoration: none;
  cursor: pointer;
  color: var(--base);
}

.versions a:hover {
  color: #3ca877;
}

.versions.expanded {
  display: block;
}

.share,
.download {
  margin: 0 2px;
}
</style>
