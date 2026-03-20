<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import { ref, onMounted } from 'vue'
import HelloWorld from './components/HelloWorld.vue'

interface Item {
  id: string | number
  item: string,
  category: string,
  purchased: boolean
}

const items = ref([])
const newItemName = ref('')
const loading = ref(false)

const API_URL = 'http://localhost:5001/api/items'

// Hämta listan (GET)
const fetchItems = async () => {
  loading.ref = true
  try {
    const res = await fetch(API_URL)
    items.value = await res.json()
  } catch (err) {
    console.error("Kunde inte hämta data", err)
  } finally {
    loading.value = false
  }
}

// Lägg till (POST)
const addItem = async () => {
  if (!newItemName.value.trim()) return

  try {
    await fetch(API_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ item: newItemName.value, category: 'Allmänt' })
    })
    newItemName.value = ''
    await fetchItems()
  } catch (err) {
    console.error("Fel vid tillägg", err)
  }
}

// Markera som köpt/ej köpt (PATCH)
const toggleStatus = async (item) => {
  try {
    await fetch(`${API_URL}/${item.id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ purchased: !item.purchased })
    })
    await fetchItems()
  } catch (err) {
    console.error("Fel vid uppdatering", err)
  }
}

// Ta bort (DELETE)
const removeItem = async (id) => {
  try {
    await fetch(`${API_URL}/${id}`, { method: 'DELETE' })
    await fetchItems()
  } catch (err) {
    console.error("Fel vid radering", err)
  }
}

onMounted(fetchItems)
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it, from within a docker container, yeeey!" />

      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
    </div>
  </header>

  <main>
    <div class="shopping-list">
    <h1>Inköpslista</h1>

    <div class="input-group">
      <input 
        v-model="newItemName" 
        @keyup.enter="addItem" 
        placeholder="Vad behöver vi handla?" 
      />
      <button @click="addItem">Lägg till</button>
    </div>

    <div v-if="loading">Laddar...</div>

    <ul v-else>
      <li v-for="item in items" :key="item.id" :class="{ purchased: item.purchased }">
        <span class="item-text" @click="toggleStatus(item)">
          {{ item.item }}
        </span>
        <button class="delete-btn" @click="removeItem(item.id)">Ta bort</button>
      </li>
    </ul>
  </div>
  </main>

  <RouterView />
</template>



<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}

.shopping-list {
  max-width: 400px;
  margin: 2rem auto;
  font-family: sans-serif;
}
.input-group {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
input {
  flex-grow: 1;
  padding: 8px;
}
ul {
  list-style: none;
  padding: 0;
}
li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #eee;
}
.item-text {
  cursor: pointer;
  flex-grow: 1;
}
.purchased .item-text {
  text-decoration: line-through;
  color: #888;
}
.delete-btn {
  background: #ff4444;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
}
</style>
