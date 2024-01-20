<script setup>
import { RouterLink, RouterView } from 'vue-router'
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <div class="app-container">
    <header>
      <div class="wrapper">
        <HelloWorld msg="Regional Pokedex Maker" />

        <nav>
          <RouterLink to="/">Home</RouterLink>
          <RouterLink to="/about">About</RouterLink>
        </nav>
      </div>
    </header>

    <RouterView />
  </div>
</template>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  width: max-content;
}

header {
  line-height: 1.5;
  width: 100%;
}

header {
align-content: center;
}

.main-content {
  display: flex;
  width: 100%;
}

@media (min-width: 768px) {
  .main-content {
    /* Larger screens can have a row-based layout */
    flex-direction: row;
  }
}
</style>
