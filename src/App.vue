<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// Native Vue SPA host for the complete original interactive pages.
// Keeping the original JS/data in same-origin pages preserves the original
// map behaviors and provides pixel-identical visuals without data conversion.
const base = import.meta.env.BASE_URL
const originalRoot = `${base}original/`
const screen = ref(`${originalRoot}index.html`)
const frame = ref(null)
const loading = ref(true)

const provinceSlugs = new Set([
  'chainat', 'chaiyaphum', 'chanthaburi', 'chiangmai', 'chiangrai',
  'kanchanaburi', 'khonkaen', 'lampang', 'maehongson', 'nan',
  'narathiwat', 'pattani', 'phatthalung', 'phetchaburi', 'phrae',
  'ranong', 'satun', 'tak', 'uttaradit', 'yala'
])

function pathForRoute() {
  const slug = new URLSearchParams(location.search).get('province')
  return slug && provinceSlugs.has(slug)
    ? `${originalRoot}${slug}/${slug}_flood_map.html`
    : `${originalRoot}index.html`
}
function navigate(url, replace = false) {
  screen.value = url
  loading.value = true
  if (replace) history.replaceState({}, '', location.pathname + urlToQuery(url))
  else history.pushState({}, '', location.pathname + urlToQuery(url))
}
function urlToQuery(url) {
  const slug = url.match(/\/original\/([a-z]+)\/\1_flood_map\.html$/)?.[1]
  return slug ? `?province=${slug}` : ''
}
function onFrameLoad() {
  loading.value = false
  try {
    const doc = frame.value.contentDocument
    const current = frame.value.contentWindow.location.pathname
    if (current !== new URL(screen.value, location.href).pathname && current.includes('/original/')) {
      screen.value = `${base}${current.split('/original/')[1] ? `original/${current.split('/original/')[1]}` : 'original/index.html'}`
      history.replaceState({}, '', location.pathname + urlToQuery(current))
    }
    // Intercept only cross-document links. All in-page interaction stays original.
    doc.addEventListener('click', e => {
      const link = e.target.closest?.('a[href]')
      if (!link || e.defaultPrevented || e.button !== 0 || e.ctrlKey || e.metaKey || e.shiftKey || e.altKey) return
      const url = new URL(link.href, location.origin)
      if (url.origin === location.origin && url.pathname.includes('/original/') && url.pathname.endsWith('.html')) {
        e.preventDefault()
        navigate(`${originalRoot}${url.pathname.split('/original/')[1]}`)
      }
    })
    // Original province switchers assign iframe-local location.href;
    // after load, reconcile browser shareable URL.
  } catch (e) {
    console.warn('Embedded view unavailable:', e)
  }
}
function onPopState() {
  screen.value = pathForRoute()
}
onMounted(() => {
  screen.value = pathForRoute()
  window.addEventListener('popstate', onPopState)
})
onUnmounted(() => window.removeEventListener('popstate', onPopState))
</script>

<template>
  <main class="app-frame" aria-label="แอปวิเคราะห์น้ำท่วมซ้ำ 20 จังหวัด">
    <iframe
      ref="frame"
      :src="screen"
      title="น้ำท่วมซ้ำ 20 จังหวัด — Interactive Dashboard"
      class="original-app"
      @load="onFrameLoad"
    />
    <div v-if="loading" class="loading" role="status">กำลังโหลดข้อมูลแผนที่…</div>
  </main>
</template>

<style>
:root { font-family: system-ui, sans-serif; }
* { box-sizing: border-box; }
html, body, #app { margin: 0; width: 100%; height: 100%; overflow: hidden; }
.app-frame { width: 100%; height: 100%; height: 100dvh; position: relative; background: white; }
.original-app { width: 100%; height: 100%; display: block; border: 0; background: white; }
.loading { position: absolute; inset: 0; display: grid; place-items: center; color: #475569; background: #fff; font-size: 15px; }
</style>
