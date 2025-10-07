<template>
  <main class="app">
    <header>
      <h1>Explorador de Países — REST Countries (Vue 3)</h1>
    </header>

    <section class="controls">
      <input v-model="search" placeholder="Buscar país por nombre..." />
      <select v-model="regionFilter">
        <option value="">Todas las regiones</option>
        <option v-for="r in regiones" :key="r" :value="r">{{ r }}</option>
      </select>
    </section>

    <section class="status">
      <p v-if="loading">Cargando países... ⏳</p>
      <p v-if="error" class="error">Error: {{ error }}</p>
      <p v-if="!loading && !error">Mostrando {{ filteredCountries.length }} países</p>
    </section>

    <section class="cards">
      <article v-for="country in paginated" :key="country.name.common" class="card">
        <img :src="country.flags.svg || country.flags.png" :alt="country.name.common + ' flag'" class="flag"/>
        <h2>{{ country.name.common }}</h2>

        <ul class="data-list">
          <li><strong>Capital:</strong> {{ country.capital ? country.capital.join(', ') : '—' }}</li>
          <li><strong>Región:</strong> {{ country.region }} / {{ country.subregion || '—' }}</li>
          <li><strong>Población:</strong> {{ formatNumber(country.population) }}</li>
          <li><strong>Idiomas:</strong> {{ formatLanguages(country.languages) }}</li>
          <li><strong>Moneda:</strong> {{ formatCurrencies(country.currencies) }}</li>
        </ul>
      </article>
    </section>

    <footer class="pagination" v-if="filteredCountries.length > pageSize">
      <button @click="prevPage" :disabled="page===1">Anterior</button>
      <span>Página {{ page }} / {{ totalPages }}</span>
      <button @click="nextPage" :disabled="page===totalPages">Siguiente</button>
    </footer>
  </main>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const API = 'https://restcountries.com/v3.1/all?fields=name,flags,capital,region,subregion,population,languages,currencies'

// reactive state
const countries = ref([])
const loading = ref(false)
const error = ref('')
const search = ref('')
const regionFilter = ref('')

// paginación simple
const page = ref(1)
const pageSize = 20

// obtener datos
async function fetchCountries() {
  loading.value = true
  error.value = ''
  try {
    const res = await fetch(API)
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    const data = await res.json()
    countries.value = data.sort((a,b) => a.name.common.localeCompare(b.name.common))
  } catch (err) {
    console.error(err)
    error.value = 'No se pudieron cargar los datos. Revisa la conexión o intenta más tarde.'
  } finally {
    loading.value = false
  }
}

onMounted(fetchCountries)

// helpers y computeds
const regiones = computed(() => {
  const set = new Set(countries.value.map(c => c.region).filter(Boolean))
  return Array.from(set).sort()
})

const filteredCountries = computed(() => {
  const q = search.value.trim().toLowerCase()
  return countries.value.filter(c => {
    const matchName = c.name.common.toLowerCase().includes(q)
    const matchRegion = regionFilter.value ? c.region === regionFilter.value : true
    return matchName && matchRegion
  })
})

const totalPages = computed(() => Math.max(1, Math.ceil(filteredCountries.value.length / pageSize)))

const paginated = computed(() => {
  // si page excede, ajusta
  if (page.value > totalPages.value) page.value = totalPages.value
  const start = (page.value - 1) * pageSize
  return filteredCountries.value.slice(start, start + pageSize)
})

function nextPage() {
  if (page.value < totalPages.value) page.value++
}
function prevPage() {
  if (page.value > 1) page.value--
}

function formatLanguages(langs) {
  if (!langs) return '—'
  return Object.values(langs).join(', ')
}

function formatCurrencies(currencies) {
  if (!currencies) return '—'
  // currencies: { USD: { name: "United States dollar", symbol: "$" }, ...}
  return Object.values(currencies).map(c => `${c.name} ${c.symbol ? '('+c.symbol+')' : ''}`).join(', ')
}

function formatNumber(n) {
  return n?.toLocaleString() ?? '—'
}
</script>

<style scoped>
.app { max-width: 1100px; margin: 0 auto; padding: 1rem; font-family: system-ui, Arial; }
header h1 { margin-bottom: 1rem; }
.controls { display:flex; gap:0.5rem; margin-bottom:1rem; }
.controls input { flex:1; padding:0.5rem; }
.controls select { padding:0.4rem; }
.status { margin-bottom:1rem; }
.error { color:crimson; }
.cards { display:grid; grid-template-columns: repeat(auto-fill,minmax(220px,1fr)); gap:1rem; }
.card { border:1px solid #ddd; padding:0.6rem; border-radius:8px; background:#fff; box-shadow: 0 1px 3px rgba(0,0,0,0.05); }
.flag { width:100%; height:120px; object-fit:cover; border-radius:4px; margin-bottom:0.5rem; }
.data-list { list-style:none; padding:0; margin:0; font-size:0.9rem; }
.pagination { display:flex; justify-content:center; gap:1rem; margin-top:1rem; align-items:center; }
button[disabled] { opacity:0.5; cursor:not-allowed; }
</style>
