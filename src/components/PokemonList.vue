<template>
  <div class="content-container">
  <div class="pokemon-list">
    <div
      v-for="pokemon in pokemons"
      :key="pokemon.id"
      class="pokemon-item"
      @click="selectPokemon(pokemon)"
    >
      <img :src="`/dexmaker-web/images/pokemons/${pokemon.id}.png`" :alt="pokemon.name">
      <p>{{ pokemon.name }}</p>
    </div>
  </div>
  <div v-if="selectedPokemon" class="pokemon-details">
    <h2>{{ selectedPokemon.name }}</h2>
    <ul>
      <h3>Abilities :</h3>
      <li v-for="ability in selectedPokemon.normal_abilities" :key="ability">
        {{ ability }}: {{ getAbilityFlavorText(ability) }}
      </li>
    </ul>
    <!-- Add more details as needed -->
  </div>
</div>
</template>

<script>
export default {
  data() {
    return {
      pokemons: [],
      abilitiesFlavorText: {},
      selectedPokemon: null,
    };
  },
  mounted() {
    fetch('/dexmaker-web/pokemon_data_sorted.json')
      .then(response => response.json())
      .then(data => {
        this.pokemons = data;
      });

    fetch('/dexmaker-web/abilities_flavor_text.json')
      .then(response => response.json())
      .then(data => {
        this.abilitiesFlavorText = data;
      });
  },
  methods: {
    selectPokemon(pokemon) {
      this.selectedPokemon = pokemon;
    },
    getAbilityFlavorText(ability) {
      console.log(this.abilitiesFlavorText[ability]?.en)
      return this.abilitiesFlavorText[ability]?.en || 'No description available.';
      
    },
  },
};
</script>



<style scoped>
.content-container {
  display: flex;
  width: 100%; /* Take the full viewport width */
}

.pokemon-list {
  width: 40%; /* Take 40% of the content-container's width */
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(100px, 1fr)); /* Adjust the minmax values as needed */
  gap: 10px; /* Space between items */
  overflow-y: auto; /* Add scrolling if content overflows */
}

.pokemon-item {
  cursor: pointer;
  text-align: center;
}

.pokemon-item img {
  width: 100%; /* Images will fill their grid item */
  height: auto;
}

.pokemon-details {
  width: 60%; /* Remaining space for details */
  /* Additional styles for details */
}
</style>


