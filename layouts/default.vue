<script setup>
import { computed, onMounted } from 'vue'
import { fetchVouchers } from '~/composables/appState'

const route = useRoute()
const showSidebar = computed(() => !['login', 'index'].includes(String(route.name ?? '')))

onMounted(async () => {
  try {
    await fetchVouchers()
  } catch (error) {
    console.error('Failed to load vouchers', error)
  }
})
</script>

<template>
  <div class="app-layout">
    <Sidebar v-if="showSidebar" />
    <main class="app-content">
      <ClientOnly>
        <slot />
      </ClientOnly>
    </main>
  </div>
</template>
