<template>
  <div class="app">
    <header>
      <h1>🐾 Animals Gallery</h1>
      <div class="tabs">
        <button :class="{ active: tab === 'cats' }" @click="tab = 'cats'">🐱 Cats</button>
        <button :class="{ active: tab === 'dogs' }" @click="tab = 'dogs'">🐶 Dogs</button>
      </div>
    </header>

    <main>
      <!-- Cats Tab -->
      <div v-if="tab === 'cats'">
        <div class="toolbar">
          <button class="refresh-btn" @click="fetchCats" :disabled="loading.cats">
            {{ loading.cats ? 'Loading…' : '🔄 Refresh' }}
          </button>
        </div>
        <p v-if="error.cats" class="error">{{ error.cats }}</p>
        <div class="grid" v-if="cats.length">
          <div v-for="cat in cats" :key="cat.id" class="card">
            <img :src="cat.url" :alt="'Cat photo'" loading="lazy" />
          </div>
        </div>
      </div>

      <!-- Dogs Tab -->
      <div v-if="tab === 'dogs'">
        <p v-if="loading.dogs" class="loading-msg">Loading breeds…</p>
        <p v-if="error.dogs" class="error">{{ error.dogs }}</p>
        <div class="grid" v-if="dogs.length">
          <div v-for="dog in dogs" :key="dog.id" class="card dog-card">
            <h3>{{ dog.attributes.name }}</h3>
            <p class="description">{{ dog.attributes.description }}</p>
            <div class="tags">
              <span v-if="dog.attributes.hypoallergenic" class="tag hypo">Hypoallergenic</span>
              <span class="tag">
                ❤️ {{ dog.attributes.life.min }}–{{ dog.attributes.life.max }} yrs
              </span>
              <span class="tag">
                ⚖️ {{ dog.attributes.male_weight.min }}–{{ dog.attributes.male_weight.max }} kg
              </span>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const tab = ref('cats')
const cats = ref([])
const dogs = ref([])
const loading = ref({ cats: false, dogs: false })
const error = ref({ cats: null, dogs: null })

async function fetchCats() {
  loading.value.cats = true
  error.value.cats = null
  try {
    const res = await fetch('https://api.thecatapi.com/v1/images/search?limit=10')
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    cats.value = await res.json()
  } catch (e) {
    error.value.cats = 'Failed to load cats: ' + e.message
  } finally {
    loading.value.cats = false
  }
}

async function fetchDogs() {
  loading.value.dogs = true
  error.value.dogs = null
  try {
    const res = await fetch('https://dogapi.dog/api/v2/breeds')
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    const json = await res.json()
    dogs.value = json.data
  } catch (e) {
    error.value.dogs = 'Failed to load dogs: ' + e.message
  } finally {
    loading.value.dogs = false
  }
}

onMounted(() => {
  fetchCats()
  fetchDogs()
})
</script>

<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: system-ui, sans-serif;
  background: #f0f4f8;
  color: #333;
}

.app {
  max-width: 1200px;
  margin: 0 auto;
  padding: 24px 16px;
}

header {
  text-align: center;
  margin-bottom: 28px;
}

h1 {
  font-size: 2rem;
  margin-bottom: 16px;
  color: #1a1a2e;
}

.tabs {
  display: inline-flex;
  gap: 8px;
  background: #dde4ee;
  padding: 4px;
  border-radius: 32px;
}

.tabs button {
  padding: 10px 32px;
  border: none;
  border-radius: 28px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: 500;
  background: transparent;
  transition: background 0.2s, color 0.2s;
}

.tabs button.active {
  background: #fff;
  color: #4a90d9;
  box-shadow: 0 2px 6px rgba(0,0,0,0.12);
}

.toolbar {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 16px;
}

.refresh-btn {
  padding: 8px 20px;
  background: #4a90d9;
  color: #fff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.95rem;
  transition: opacity 0.2s;
}

.refresh-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 20px;
}

.card {
  background: #fff;
  border-radius: 14px;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0,0,0,0.08);
  transition: transform 0.2s, box-shadow 0.2s;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 6px 20px rgba(0,0,0,0.12);
}

.card img {
  width: 100%;
  height: 200px;
  object-fit: cover;
  display: block;
  background: #eee;
}

.dog-card {
  padding: 18px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.dog-card h3 {
  font-size: 1.05rem;
  color: #1a1a2e;
}

.description {
  font-size: 0.85rem;
  color: #666;
  line-height: 1.5;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: auto;
}

.tag {
  background: #e8f0fe;
  color: #1a73e8;
  font-size: 0.75rem;
  padding: 3px 10px;
  border-radius: 12px;
  white-space: nowrap;
}

.tag.hypo {
  background: #e6f4ea;
  color: #1e8e3e;
}

.error {
  color: #c0392b;
  margin: 16px 0;
  text-align: center;
}

.loading-msg {
  text-align: center;
  color: #888;
  margin: 40px 0;
  font-size: 1.1rem;
}
</style>
