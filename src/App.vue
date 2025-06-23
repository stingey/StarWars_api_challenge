<template>
  <div class="p-4" style="min-height: 500px;">
    <h1 class="mb-4 fw-bold fs-3">Star Wars Explorer</h1>

    <!-- Category Buttons -->
    <div class="d-flex flex-wrap justify-content-center gap-2 mb-4">
      <button
        v-for="cat in categories"
        :key="cat"
        type="button"
        :class="[
          'btn', 'text-capitalize',
          selectedCategory === cat ? 'btn-primary' : 'btn-outline-primary'
        ]"
        @click="selectCategory(cat)"
      >
        {{ cat }}
      </button>
    </div>

    <!-- Loading / Error -->
    <div v-if="loading" class="mb-3">Loading...</div>
    <div v-else-if="error" class="text-danger mb-3">Error: {{ error }}</div>

    <!-- Item List -->
    <div v-else-if="items.length && !selectedItem">
      <p v-for="item in items" :key="item.uid" class="mb-2">
        <a href="#" class="text-primary text-decoration-underline" @click.prevent="fetchDetails(item.url)">
          {{ item.name || item.title }}
        </a>
      </p>
    </div>

    <!-- Item Details -->
    <div v-else-if="selectedItem">
      <button
        type="button"
        class="btn btn-secondary mb-3"
        @click="selectedItem = null"
      >
        ← Back
      </button>
      <p v-for="(value, key) in selectedItem" :key="key">
        <strong class="text-capitalize">{{ key.replace(/_/g, ' ') }}:</strong> {{ value }}
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const categories = ['people', 'planets', 'species', 'starships', 'vehicles']
const selectedCategory = ref('people') // Default category
const items = ref([])
const selectedItem = ref(null)
const loading = ref(false)
const error = ref(null)

function selectCategory(cat) {
  if (selectedCategory.value !== cat) {
    selectedCategory.value = cat
    fetchList()
  }
}

async function fetchList() {
  selectedItem.value = null
  loading.value = true
  error.value = null

  try {
    const res = await fetch(`https://www.swapi.tech/api/${selectedCategory.value}`)
    if (!res.ok) throw new Error('Failed to fetch list')
    const data = await res.json()
    items.value = data.results || []
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

async function fetchDetails(url) {
  loading.value = true
  error.value = null

  try {
    const res = await fetch(url)
    if (!res.ok) throw new Error('Failed to fetch details')
    const data = await res.json()
    selectedItem.value = data.result.properties
  } catch (err) {
    error.value = err.message
  } finally {
    loading.value = false
  }
}

// Fetch default category on load
onMounted(fetchList)
</script>
