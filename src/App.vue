<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

interface Item {
  id: string | number
  item: string
  category: string
  purchased: boolean
}

const items = ref<Item[]>([])
const newItemName = ref('')
const newItemCategory = ref('')
const loading = ref(false)
const filterCategory = ref<string | null>(null)

const API_URL = 'http://localhost:5001/api/items'

const categoryColors: Record<string, string> = {
  'Mat': '#FF6B6B',
  'Dryck': '#4ECDC4',
  'Rengöring': '#45B7D1',
  'Husgeråd': '#96CEB4',
  'Allmänt': '#A78BFA',
  'Personlig hygien': '#FD79A8',
}

function getCategoryColor(cat: string): string {
  return categoryColors[cat] || '#A78BFA'
}

const categories = computed(() => {
  const cats = new Set(items.value.map(i => i.category).filter(Boolean))
  return Array.from(cats)
})

const filteredItems = computed(() => {
  if (!filterCategory.value) return items.value
  return items.value.filter(i => i.category === filterCategory.value)
})

const purchasedCount = computed(() => items.value.filter(i => i.purchased).length)

const fetchItems = async () => {
  loading.value = true
  try {
    const res = await fetch(API_URL)
    items.value = await res.json()
  } catch (err) {
    console.error('Kunde inte hämta data', err)
  } finally {
    loading.value = false
  }
}

const addItem = async () => {
  if (!newItemName.value.trim()) return
  try {
    await fetch(API_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        item: newItemName.value,
        category: newItemCategory.value || 'Allmänt',
      }),
    })
    newItemName.value = ''
    newItemCategory.value = ''
    await fetchItems()
  } catch (err) {
    console.error('Fel vid tillägg', err)
  }
}

const toggleStatus = async (item: Item) => {
  try {
    await fetch(`${API_URL}/${item.id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ purchased: !item.purchased }),
    })
    await fetchItems()
  } catch (err) {
    console.error('Fel vid uppdatering', err)
  }
}

const removeItem = async (id: string | number) => {
  try {
    await fetch(`${API_URL}/${id}`, { method: 'DELETE' })
    await fetchItems()
  } catch (err) {
    console.error('Fel vid radering', err)
  }
}

onMounted(fetchItems)
</script>

<template>
  <div class="app-shell">
    <!-- Header -->
    <header class="app-header">
      <div class="header-inner">
        <div class="header-title">
          <span class="header-icon">🛒</span>
          <h1>Inköpslista</h1>
        </div>
        <div class="header-stats" v-if="items.length > 0">
          <span class="stat-pill">{{ purchasedCount }}/{{ items.length }}</span>
        </div>
      </div>
    </header>

    <!-- Add Item Bar -->
    <div class="add-bar">
      <div class="add-bar-inner">
        <input
          v-model="newItemName"
          class="add-input"
          placeholder="Lägg till vara..."
          @keyup.enter="addItem"
        />
        <input
          v-model="newItemCategory"
          class="add-input category-input"
          placeholder="Kategori"
          @keyup.enter="addItem"
        />
        <button class="add-btn" @click="addItem" :disabled="!newItemName.trim()">
          <span class="add-btn-icon">+</span>
        </button>
      </div>
    </div>

    <!-- Category Filter Chips -->
    <div class="filter-bar" v-if="categories.length > 0">
      <button
        class="chip"
        :class="{ active: filterCategory === null }"
        @click="filterCategory = null"
      >Alla</button>
      <button
        v-for="cat in categories"
        :key="cat"
        class="chip"
        :class="{ active: filterCategory === cat }"
        :style="filterCategory === cat ? `background: ${getCategoryColor(cat)}22; border-color: ${getCategoryColor(cat)}; color: ${getCategoryColor(cat)}` : ''"
        @click="filterCategory = filterCategory === cat ? null : cat"
      >{{ cat }}</button>
    </div>

    <!-- List -->
    <main class="list-area">
      <!-- Loading -->
      <div v-if="loading" class="loading-state">
        <div class="spinner"></div>
      </div>

      <!-- Empty -->
      <div v-else-if="filteredItems.length === 0" class="empty-state">
        <span class="empty-icon">🛍️</span>
        <p>Listan är tom</p>
      </div>

      <!-- Items -->
      <transition-group v-else name="list" tag="ul" class="items-list">
        <li
          v-for="item in filteredItems"
          :key="item.id"
          class="item-card"
          :class="{ 'item-purchased': item.purchased }"
        >
          <button
            class="check-btn"
            :class="{ checked: item.purchased }"
            :style="item.purchased ? `background: ${getCategoryColor(item.category)}; border-color: ${getCategoryColor(item.category)}` : `border-color: ${getCategoryColor(item.category)}55`"
            @click="toggleStatus(item)"
          >
            <svg v-if="item.purchased" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="3">
              <polyline points="20 6 9 17 4 12" />
            </svg>
          </button>

          <div class="item-body">
            <span class="item-name">{{ item.item }}</span>
            <span
              class="item-cat-badge"
              :style="`background: ${getCategoryColor(item.category)}22; color: ${getCategoryColor(item.category)}`"
            >{{ item.category }}</span>
          </div>

          <button class="delete-btn" @click="removeItem(item.id)">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="3 6 5 6 21 6" />
              <path d="M19 6l-1 14a2 2 0 01-2 2H8a2 2 0 01-2-2L5 6" />
              <path d="M10 11v6M14 11v6" />
              <path d="M9 6V4h6v2" />
            </svg>
          </button>
        </li>
      </transition-group>
    </main>
  </div>
</template>

<style scoped>
/* ─── Shell ─────────────────────────────────────────────────────── */
.app-shell {
  min-height: 100vh;
  background: linear-gradient(145deg, #0f0c29, #302b63, #24243e);
  display: flex;
  flex-direction: column;
  font-family: 'Inter', sans-serif;
  max-width: 480px;
  margin: 0 auto;
}

/* ─── Header ─────────────────────────────────────────────────────── */
.app-header {
  padding: 3rem 1.5rem 1.5rem;
}
.header-inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.header-title {
  display: flex;
  align-items: center;
  gap: 0.6rem;
}
.header-icon {
  font-size: 1.8rem;
}
.app-header h1 {
  font-size: 1.75rem;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.5px;
  margin: 0;
}
.header-stats {
  display: flex;
}
.stat-pill {
  background: rgba(255,255,255,0.12);
  border: 1px solid rgba(255,255,255,0.2);
  color: rgba(255,255,255,0.85);
  border-radius: 999px;
  padding: 0.25rem 0.75rem;
  font-size: 0.8rem;
  font-weight: 600;
  letter-spacing: 0.3px;
}

/* ─── Add Bar ─────────────────────────────────────────────────────── */
.add-bar {
  padding: 0 1.5rem 1rem;
}
.add-bar-inner {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 16px;
  padding: 0.5rem;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
}
.add-input {
  background: transparent;
  border: none;
  outline: none;
  color: #fff;
  font-size: 0.95rem;
  flex: 1;
  padding: 0.5rem 0.4rem;
  min-width: 0;
}
.add-input::placeholder {
  color: rgba(255, 255, 255, 0.35);
}
.category-input {
  flex: 0 0 30%;
  border-left: 1px solid rgba(255, 255, 255, 0.12);
  padding-left: 0.7rem;
}
.add-btn {
  background: linear-gradient(135deg, #a78bfa, #7c3aed);
  border: none;
  border-radius: 11px;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
  flex-shrink: 0;
  box-shadow: 0 4px 14px rgba(124, 58, 237, 0.45);
}
.add-btn:hover:not(:disabled) {
  transform: scale(1.05);
  box-shadow: 0 6px 20px rgba(124, 58, 237, 0.6);
}
.add-btn:disabled {
  opacity: 0.4;
  cursor: default;
}
.add-btn-icon {
  color: #fff;
  font-size: 1.5rem;
  line-height: 1;
  font-weight: 300;
}

/* ─── Filter Chips ─────────────────────────────────────────────────── */
.filter-bar {
  display: flex;
  gap: 0.5rem;
  padding: 0 1.5rem 1.25rem;
  overflow-x: auto;
  scrollbar-width: none;
}
.filter-bar::-webkit-scrollbar { display: none; }
.chip {
  border-radius: 999px;
  border: 1px solid rgba(255, 255, 255, 0.18);
  background: rgba(255, 255, 255, 0.07);
  color: rgba(255, 255, 255, 0.6);
  padding: 0.3rem 0.85rem;
  font-size: 0.78rem;
  font-weight: 600;
  white-space: nowrap;
  cursor: pointer;
  transition: all 0.2s ease;
  letter-spacing: 0.2px;
}
.chip:hover {
  background: rgba(255, 255, 255, 0.14);
}
.chip.active {
  background: rgba(167, 139, 250, 0.2);
  border-color: #a78bfa;
  color: #a78bfa;
}

/* ─── List Area ─────────────────────────────────────────────────────── */
.list-area {
  flex: 1;
  padding: 0 1.5rem 2rem;
}
.items-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
}

/* ─── Item Card ─────────────────────────────────────────────────────── */
.item-card {
  display: flex;
  align-items: center;
  gap: 0.85rem;
  background: rgba(255, 255, 255, 0.07);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 14px;
  padding: 0.9rem 1rem;
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  transition: all 0.2s ease;
}
.item-card:hover {
  background: rgba(255, 255, 255, 0.11);
  border-color: rgba(255, 255, 255, 0.18);
  transform: translateY(-1px);
}
.item-purchased {
  opacity: 0.45;
}

/* ─── Check Button ─────────────────────────────────────────────────── */
.check-btn {
  width: 26px;
  height: 26px;
  border-radius: 50%;
  border: 2px solid rgba(255, 255, 255, 0.3);
  background: transparent;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  transition: all 0.2s ease;
}
.check-btn:hover {
  transform: scale(1.1);
}
.check-btn.checked {
  border-color: transparent;
}
.check-btn svg {
  width: 13px;
  height: 13px;
}

/* ─── Item Body ─────────────────────────────────────────────────────── */
.item-body {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  min-width: 0;
}
.item-name {
  color: #ffffff;
  font-size: 0.97rem;
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.item-purchased .item-name {
  text-decoration: line-through;
  color: rgba(255, 255, 255, 0.5);
}
.item-cat-badge {
  display: inline-block;
  font-size: 0.7rem;
  font-weight: 600;
  letter-spacing: 0.3px;
  padding: 0.1rem 0.55rem;
  border-radius: 999px;
  align-self: flex-start;
  text-transform: uppercase;
}

/* ─── Delete Button ─────────────────────────────────────────────────── */
.delete-btn {
  background: transparent;
  border: none;
  cursor: pointer;
  color: rgba(255, 255, 255, 0.25);
  padding: 0.3rem;
  display: flex;
  align-items: center;
  border-radius: 8px;
  transition: all 0.2s ease;
  flex-shrink: 0;
}
.delete-btn:hover {
  color: #ff6b6b;
  background: rgba(255, 107, 107, 0.12);
}
.delete-btn svg {
  width: 17px;
  height: 17px;
}

/* ─── States ─────────────────────────────────────────────────────── */
.loading-state {
  display: flex;
  justify-content: center;
  padding: 3rem 0;
}
.spinner {
  width: 36px;
  height: 36px;
  border: 3px solid rgba(255,255,255,0.12);
  border-top-color: #a78bfa;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}
@keyframes spin {
  to { transform: rotate(360deg); }
}
.empty-state {
  text-align: center;
  padding: 4rem 0;
  color: rgba(255, 255, 255, 0.35);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
}
.empty-icon {
  font-size: 2.5rem;
}
.empty-state p {
  font-size: 0.95rem;
}

/* ─── List Transition ─────────────────────────────────────────────── */
.list-enter-active,
.list-leave-active {
  transition: all 0.25s ease;
}
.list-enter-from {
  opacity: 0;
  transform: translateX(-12px);
}
.list-leave-to {
  opacity: 0;
  transform: translateX(12px);
}
</style>