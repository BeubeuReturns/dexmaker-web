<template>
  <div class="content-container">
    <div class="pokemon-list">
      <div
        v-for="pokemon in pokemons"
        :key="pokemon.id"
        class="pokemon-item"
        @click="selectPokemon(pokemon)"
      >
        <img :src="`/dexmaker-web/images/pokemons/${pokemon.id}.png`" :alt="pokemon.name" />
        <p>{{ formatName(pokemon.name) }}</p>
      </div>
    </div>
    <div v-if="selectedPokemon" class="pokemon-details">
      <h2>{{ formatName(selectedPokemon.name) }}</h2>
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
            :style="{ width: getBarWidth(value), backgroundColor: getStatColor(key) }"
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
  },
  methods: {
    selectPokemon(pokemon) {
      this.selectedPokemon = pokemon
    },
    getAbilityFlavorText(ability) {
      console.log(this.abilitiesFlavorText[ability]?.en)
      return this.abilitiesFlavorText[ability]?.en || 'No description available.'
    },
    getStatColor(stat) {
      const colors = {
        hp: 'yellow',
        attack: 'orange',
        defense: 'red',
        'special-attack': 'green',
        'special-defense': 'blue',
        speed: 'purple'
      }
      return colors[stat] || 'grey'
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
    }
  }
}
</script>

<style scoped>
.pokemon-stats .stat {
  display: flex;
  align-items: center;
}

.pokemon-stats .stat span {
  margin-right: 5px;
}

.bar-container {
  height: 10px; /* Adjust as needed */
  background-color: transparent; /* Light grey background for the bar */
  border-radius: 2px; /* Rounded corners for the bar container */
}


.stat {
  display: grid;
  grid-template-columns: minmax(auto, 100px) minmax(auto, 50px) auto; /* Adjust as needed */
  align-items: center;
  gap: 0.5rem;
}

.stat-name,
.stat-value {
  text-align: right; /* Align text to the right */
}

.bar-container {
  width: 100%; /* Take up remaining space */
  background-color: transparent;
  border-radius: 5px;
  overflow: hidden; /* Ensures inner bar respects border-radius */
}

.bar {
  width: 0; /* Initial width */
  height: 100%;
  border-radius: 5px; /* Optionally rounded corners for the bar itself */
  transition: width 0.5s ease; /* Smooth transition for bar width */
}

.content-container {
  display: flex;
  flex-direction: column; /* Stack elements vertically on smaller screens */
  width: 98vw; /* Ensure it spans the full viewport width */
  min-height: 100vh; /* Use at least the full viewport height */
}

.pokemon-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(96px, 1fr));
  padding: 0.5rem; /* Add padding around the grid */
  width: 100%; /* Use full width on smaller screens */
}

.pokemon-details {
  width: 100%; /* Use full width on smaller screens */
  padding: 1rem; /* Add padding for consistency */
}

/* Media query for larger screens */
@media (min-width: 768px) {
  .content-container {
    flex-direction: row; /* Side-by-side layout on larger screens */
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
  width: 40%; /* Fill the remaining 40% of the page */
}

.hidden-ability {
  font-style: italic;
}
</style>
