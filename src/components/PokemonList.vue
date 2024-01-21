<template>
  <div class="selection-bar">
    
    Selected: {{ selectedPokemonIds.length }}
    <button @click="deselectAllPokemons">Deselect All</button>
    <label>
      <input type="checkbox" v-model="selectByEvolutionLine" @change="updateFinalStages">
      Select by Evolution Line
    </label>
    <button @click="exportSelectedPokemons">Export Selected</button>
  </div>
  <div class="content-container">
    <div class="pokemon-list">
  <div
    v-for="pokemon in displayPokemons"
    :key="pokemon.id"
    :class="{ 'selected': isSelected(pokemon.id) }"
    class="pokemon-item"
    @click="handlePokemonSelection(pokemon)"
  >
        <img :src="`/dexmaker-web/images/pokemons/${pokemon.id}.png`" :alt="formatName(pokemon.name)" />
        <p>{{ formatName(pokemon.name) }}</p>
      </div>
    </div>
    <div v-if="selectedPokemon" class="pokemon-details">
      <h2>{{ formatName(selectedPokemon.name) }}</h2>
      <div>
        <img 
          v-for="type in selectedPokemon.types" 
          :key="type" 
          :src="`/dexmaker-web/images/types/${formatName(type)}.png`" 
          :alt="type" 
          class="type-icon" 
        />
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
      <h3>Stats:</h3>
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
      selectedPokemon: null,
      evolutionChains: {},
      selectByEvolutionLine: false
    }
  },
  watch: {
  selectedPokemonIds(newVal) {
    localStorage.setItem('selectedPokemonIds', JSON.stringify(newVal));
  }
},

computed: {
  displayPokemons() {
    if (this.selectByEvolutionLine) {
      // Iterate over the evolution chains and accumulate all final forms' IDs
      const finalFormIds = new Set();
      Object.values(this.evolutionChains).forEach(chain => {
        Object.values(chain).forEach(details => {
          details.final_forms.forEach(finalFormName => {
            const pokemon = this.pokemons.find(p => p.name === finalFormName);
            if (pokemon) {
              finalFormIds.add(pokemon.id);
            }
          });
        });
      });

      // Filter the pokemons to include only those that are final evolutions
      return this.pokemons.filter(pokemon => finalFormIds.has(pokemon.id));
    }

    // If not selecting by evolution line, return all pokemons
    return this.pokemons;
  },
  finalStagePokemons() {
    if (!this.selectByEvolutionLine) {
      return this.pokemons;
    }
    return this.pokemons.filter(pokemon => {
      const chainDetails = this.findEvolutionChainDetails(pokemon.name);
      return chainDetails && chainDetails.final_forms.includes(pokemon.name);
    });
  },
},


mounted() {
  // Fetching the Pokémon data
  fetch('/dexmaker-web/pokemon_data_sorted.json')
    .then(response => response.json())
    .then(data => {
      this.pokemons = data.map(pokemon => ({
        ...pokemon,
        evolutionChainId: this.findEvolutionChainId(pokemon.name)
      }));
      console.log("Pokémons data loaded:", this.pokemons);
    });

  // Fetching the abilities flavor text
  fetch('/dexmaker-web/abilities_flavor_text.json')
    .then(response => response.json())
    .then(data => {
      this.abilitiesFlavorText = data;
      console.log("Abilities flavor text loaded:", this.abilitiesFlavorText);
    });

  // Fetching evolution chain data
  fetch('/dexmaker-web/evolution_chains.json')
    .then(response => response.json())
    .then(data => {
      this.evolutionChains = data;
      console.log("Evolution chains loaded:", this.evolutionChains);
    });

  // Load selected Pokémon IDs from local storage
  const savedSelections = localStorage.getItem('selectedPokemonIds');
  if (savedSelections) {
    this.selectedPokemonIds = JSON.parse(savedSelections);
    console.log("Selected Pokémon IDs loaded from localStorage:", this.selectedPokemonIds);
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
    
  deselectAllPokemons() {
    this.selectedPokemonIds = [];
  },
  toggleEvolutionLine(pokemonName) {
  const chainDetails = this.findEvolutionChainDetails(pokemonName);
  if (chainDetails) {
    let selectedIds;
    if (this.selectByEvolutionLine) {
      // If we are selecting by evolution line, select only the final forms
      selectedIds = chainDetails.final_forms.map(finalFormName => {
        const finalFormPokemon = this.pokemons.find(pokemon => pokemon.name === finalFormName);
        return finalFormPokemon ? finalFormPokemon.id : null;
      }).filter(id => id !== null); // Filter out any nulls
    } else {
      // If we are not selecting by evolution line, select the entire chain
      selectedIds = chainDetails.full_chain.map(pokemonName => {
        const pokemon = this.pokemons.find(pokemon => pokemon.name === pokemonName[0]);
        return pokemon ? pokemon.id : null;
      }).filter(id => id !== null); // Filter out any nulls
    }
    this.selectedPokemonIds = selectedIds;
    localStorage.setItem('selectedPokemonIds', JSON.stringify(this.selectedPokemonIds));
  }
},


findEvolutionChain(pokemonName) {
  for (const chain of Object.values(this.evolutionChains)) {
    for (const details of Object.values(chain)) {
      const fullChainNames = details.full_chain.map(item => item[0]);
      if (fullChainNames.includes(pokemonName)) {
        // Return only the IDs from the full chain
        return details.full_chain.map(item => item[1]);
      }
    }
  }
  return null;
},

    isSelected(pokemonId) {
    return this.selectedPokemonIds.includes(pokemonId);
  },

  isFinalStage(pokemon) {
    console.log("Checking if final stage for:", pokemon.name);
    const chainId = this.findEvolutionChainId(pokemon.name);
    if (chainId) {
      const chainDetails = this.evolutionChains[chainId];
      for (const details of Object.values(chainDetails)) {
        if (details.final_forms.includes(pokemon.name)) {
          console.log("Is final stage:", true);
          return true;
        }
      }
    }
    console.log("Is final stage:", false);
    return false;
  },

  handlePokemonSelection(pokemon) {
    if (this.selectByEvolutionLine && this.isFinalStage(pokemon)) {
      if (this.isSelected(pokemon.id)) {
        // If the Pokémon is already selected, deselect the entire chain
        this.deselectEntireChain(pokemon);
      } else {
        // If the Pokémon is not selected, select the entire chain
        this.selectEntireChain(pokemon);
      }
    } else {
      // For normal view, just toggle the selection
      this.togglePokemonSelection(pokemon);
    }
  },




selectEntireChain(pokemon) {
    console.log("Selecting entire chain for:", pokemon.name);

    // Find the evolution chain based on the selected Pokémon's evolutionChainId
    const chainId = this.findEvolutionChainId(pokemon.name);
    if (chainId) {
      console.log("Found chain details for:", pokemon.name);

      // Access the correct chain details using chainId and the species name
      const speciesChain = this.evolutionChains[chainId];
      for (const [, details] of Object.entries(speciesChain)) {
        if (details.final_forms.includes(pokemon.name)) {
          details.full_chain.forEach(([, id]) => {
            if (!this.selectedPokemonIds.includes(id)) {
              this.selectedPokemonIds.push(id);
              console.log("Adding to selection:", id);
            }
          });
          break; // Exit loop after finding the correct chain
        }
      }
    } else {
      console.log("No chain details found for:", pokemon.name);
    }

    localStorage.setItem('selectedPokemonIds', JSON.stringify(this.selectedPokemonIds));
    console.log("Updated selectedPokemonIds:", this.selectedPokemonIds);
  },


  togglePokemonSelection(pokemon) {
    const index = this.selectedPokemonIds.indexOf(pokemon.id);
    if (this.selectByEvolutionLine && this.isFinalStage(pokemon)) {
      if (index > -1) {
        // Deselect the entire chain if the Pokémon is already selected
        this.deselectEntireChain(pokemon);
      } else {
        // Select the entire chain if the Pokémon is not selected
        this.selectEntireChain(pokemon);
      }
    } else {
      // For the normal view or if the Pokémon is not a final stage
      if (index > -1) {
        this.selectedPokemonIds.splice(index, 1); // Deselect
      } else {
        this.selectedPokemonIds.push(pokemon.id); // Select
      }
    }
    localStorage.setItem('selectedPokemonIds', JSON.stringify(this.selectedPokemonIds));
  },

  deselectSinglePokemon(pokemonId) {
    const index = this.selectedPokemonIds.indexOf(pokemonId);
    if (index > -1) {
      this.selectedPokemonIds.splice(index, 1); // Deselect
      localStorage.setItem('selectedPokemonIds', JSON.stringify(this.selectedPokemonIds));
    }
  },

  deselectEntireChain(pokemon) {
    const chainId = this.findEvolutionChainId(pokemon.name);
    if (chainId) {
      const speciesChain = this.evolutionChains[chainId];
      for (const [, details] of Object.entries(speciesChain)) {
        details.full_chain.forEach(([, id]) => {
          const index = this.selectedPokemonIds.indexOf(id);
          if (index > -1) {
            this.selectedPokemonIds.splice(index, 1);
          }
        });
      }
      localStorage.setItem('selectedPokemonIds', JSON.stringify(this.selectedPokemonIds));
    }
  },

  exportSelectedPokemons() {
    const selectedPokemons = this.pokemons.filter(pokemon => this.selectedPokemonIds.includes(pokemon.id));
    
    // Convert selectedPokemons to JSON
    const selectedPokemonsJson = JSON.stringify(selectedPokemons, null, 2);

    // Option 1: Log the JSON to the console
    console.log("Exporting Selected Pokémons:", selectedPokemonsJson);

    // Option 2: Trigger a file download with the JSON data
    const blob = new Blob([selectedPokemonsJson], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.download = 'selected-pokemons.json';
    link.click();
    URL.revokeObjectURL(url);
  },

  findEvolutionChainId(pokemonName) {
    console.log("Finding evolution chain ID for:", pokemonName);

    for (const [chainId, speciesData] of Object.entries(this.evolutionChains)) {
      for (const [, details] of Object.entries(speciesData)) {
        const namesInChain = details.full_chain.map(entry => entry[0].toLowerCase());
        if (namesInChain.includes(pokemonName.toLowerCase())) {
          console.log("Found evolution chain ID for " + pokemonName + ": " + chainId);
          return chainId;
        }
      }
    }

    console.log("No evolution chain ID found for:", pokemonName);
    return null;
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
  border: 2px solid rgb(30, 255, 0); /* Or any style to highlight */
  border-radius: 5px;
}

.type-icon {
  width: 80px; /* Adjust as needed */
  height: 16px; /* Adjust as needed */
  margin-right: 5px;
}
.selection-bar {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  /* Add more styling as needed */
}

</style>
