<template>
  <div class="content-container">
    <div class="pokemon-list">
      
      <div
  v-for="pokemon in pokemons"
  :key="pokemon.id"
  :class="{ 'selected': selectedPokemonIds.includes(pokemon.id) }"
  class="pokemon-item"
  @click="togglePokemonSelection(pokemon)"
>
  <img :src="`/dexmaker-web/images/pokemons/${pokemon.id}.png`" :alt="pokemon.name" />
  <p>{{ formatName(pokemon.name) }}</p>
  <div v-if="selectedPokemonIds.includes(pokemon.id)">
  </div>
</div>
    </div>
    <div v-if="selectedPokemon" class="pokemon-details">
      <h2>{{ formatName(selectedPokemon.name) }}</h2>
      <div>
    <img v-for="type in selectedPokemon.types" :key="type" :src="`/dexmaker-web/images/types/${formatName(type)}.png`" :alt="type" class="type-icon" />
  </div>
      <ul>
        <h3>Abilities:</h3>
        <li v-for="ability in selectedPokemon.normal_abilities" :key="ability">
          {{ formatAbility(ability) }}: {{ getAbilityFlavorText(ability) }}
        </li>
        <li
          v-for="ability in selectedPokemon.hidden_abilities"
          :key="ability"
          class="hidden-ability"
        >
          {{ formatAbility(ability) }}: {{ getAbilityFlavorText(ability) }}
        </li>
      </ul>
      <h3>Stats :</h3>
      <div class="stat" v-for="(value, key) in selectedPokemon.stats" :key="key">

        <span class="stat-name">{{ getStatName(key) }}:</span>
        <span class="stat-value">{{ value }}</span>
        <div class="bar-container">
          <div
          class="bar"
  :style="{ width: getBarWidth(value), backgroundColor: getStatColor(value) }"
          ></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      pokemons: [],
      selectedPokemonIds: [],
      abilitiesFlavorText: {},
      selectedPokemon: null
    }
  },
  mounted() {
    fetch('/dexmaker-web/pokemon_data_sorted.json')
      .then((response) => response.json())
      .then((data) => {
        this.pokemons = data
      })

    fetch('/dexmaker-web/abilities_flavor_text.json')
      .then((response) => response.json())
      .then((data) => {
        this.abilitiesFlavorText = data
      })
      // Load from local storage
  const savedSelections = localStorage.getItem('selectedPokemonIds');
  if (savedSelections) {
    this.selectedPokemonIds = JSON.parse(savedSelections);
  }
  },
  methods: {
    selectPokemon(pokemon) {
      this.selectedPokemon = pokemon
    },
    getAbilityFlavorText(ability) {
      console.log(this.abilitiesFlavorText[ability]?.en)
      return this.abilitiesFlavorText[ability]?.en || 'No description available.'
    },
    getStatColor(value) {
    if (value >= 175) return "blue";
    if (value >= 150) return "teal";
    if (value >= 125) return "lime";
    if (value >= 90) return "green";
    if (value >= 75) return "yellow";
    if (value >= 50) return "orange";
    return "red";
  },
    getBarWidth(value) {
      const maxStatValue = 200
      const widthPercentage = (value / maxStatValue) * 100
      return `${widthPercentage}%`
    },
    getStatName(key) {
      const names = {
        hp: 'HP',
        attack: 'Attack',
        defense: 'Defense',
        'special-attack': 'Spe Atk',
        'special-defense': 'Spe Def',
        speed: 'Speed'
      }
      return names[key] || key
    },
    formatName(name) {
      return name.charAt(0).toUpperCase() + name.slice(1)
    },
    formatAbility(ability) {
      return ability
        .split('-')
        .map((part) => part.charAt(0).toUpperCase() + part.slice(1))
        .join(' ')
    },
    
    togglePokemonSelection(pokemon) {
      this.selectedPokemon = pokemon; 
    const index = this.selectedPokemonIds.indexOf(pokemon.id);
    if (index > -1) {
      this.selectedPokemonIds.splice(index, 1); // Deselect
    } else {
      this.selectedPokemonIds.push(pokemon.id); // Select
    }
    localStorage.setItem('selectedPokemonIds', JSON.stringify(this.selectedPokemonIds));
    },
  exportSelectedPokemons() {
    const selectedPokemons = this.pokemons.filter(pokemon => this.selectedPokemonIds.includes(pokemon.id));
    // Export logic for `selectedPokemons`
  },

  
  }
}
</script>

<style scoped>
.content-container {
  display: flex;
  flex-direction: column; /* Stack elements vertically on smaller screens */
  width: 98vw; /* Ensure it spans the full viewport width */
  min-height: 100vh; /* Use at least the full viewport height */
  align-items: center; /* Center the content horizontally */
}

.pokemon-list {
  overflow-y: auto; /* Enables vertical scrolling */
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(96px, 1fr));
  gap: 0.5rem;
  padding: 0.5rem; /* Add padding around the grid */
  width: 60%; /* Adjusted for side-by-side layout */
  height: calc(100vh - 1rem); /* Full height minus padding, adjust as needed */
}

.pokemon-details {
  width: 40%; /* Use the remaining 40% of the content-container's width */
  padding: 1rem; /* Add padding for consistency */
}

/* Media query for larger screens */
@media (min-width: 768px) {
  .content-container {
    flex-direction: row; /* Side-by-side layout on larger screens */
  }
  
  .pokemon-list, .pokemon-details {
    height: calc(100vh - 2rem); /* Adjust height when side by side */
  }
}

.pokemon-item {
  cursor: pointer;
  text-align: center;
}

.pokemon-item img {
  width: 80%;
  height: auto;
}

.pokemon-details {
  overflow-y: auto; /* Scroll for details if needed */
}

.hidden-ability {
  font-style: italic;
}

/* Rest of your styles for stat bars etc. */
.stat {
  display: grid;
  grid-template-columns: minmax(auto, 100px) minmax(auto, 50px) auto;
  align-items: center;
  gap: 0.5rem;
}

.stat-name,
.stat-value {
  text-align: right;
}

.bar-container {
  width: 100%;
  background-color: transparent;
  border-radius: 2px;
  overflow: hidden;
  height: 5px;
}

.bar {
  height: 100%;
  border-radius: 5px;
  transition: width 0.5s ease;
}

.selected {
  border: 2px solid blue; /* Or any style to highlight */
}

.type-icon {
  width: 80px; /* Adjust as needed */
  height: 16px; /* Adjust as needed */
  margin-right: 5px;
}

</style>
