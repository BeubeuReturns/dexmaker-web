<template>
    <div>
      <ul>
        <li v-for="pokemon in pokemons" :key="pokemon.name">
          {{ pokemon.name }}
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        pokemons: []
      };
    },
    mounted() {
      fetch('https://pokeapi.co/api/v2/pokemon?limit=100')
        .then(response => response.json())
        .then(data => {
          this.pokemons = data.results;
        });
    }
  };
  </script>
  
  <style scoped>
  /* Add your styles here */
  </style>
  