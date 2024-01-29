<script setup>
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <div class="app-container">
    <header>
      <div class="wrapper">
        <h1>Regional Pokedex Maker</h1>

        <nav>
         <h2> <RouterLink to="/">Pokémon List</RouterLink>>
          <RouterLink to="/overview">Overview</RouterLink></h2>
        </nav>
      </div>
    </header>

    <RouterView />
  </div>
</template>


<style scoped>
h1{
  color:limegreen;
  font-size:xx-large;
  padding-bottom: 1%;
}

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
