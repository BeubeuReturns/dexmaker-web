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
      <h3>Hidden Ability :</h3>
      <li v-for="ability in selectedPokemon.hidden_abilities" :key="ability">
        {{ ability }}: {{ getAbilityFlavorText(ability) }}
      </li>
    </ul>
    <h3>Stats :</h3>
    <div class="stat" v-for="(value, key) in selectedPokemon.stats" :key="key">
    <span class="stat-name">{{ getStatName(key) }}:</span>
    <span class="stat-value">{{ value }}</span>
    <div class="bar-container">
      <div class="bar" :style="{ width: getBarWidth(value), backgroundColor: getStatColor(key) }"></div>
    </div>
      <span>{{ value }}</span>
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
    getStatColor(stat) {
      const colors = {
        hp: 'yellow',
        attack: 'orange',
        defense: 'red',
        'special-attack': 'green',
        'special-defense': 'blue',
        speed: 'purple'
      };
      return colors[stat] || 'grey';
    },
    getBarWidth(value) {
      const maxStatValue = 200;
      const widthPercentage = (value / maxStatValue) * 100;
      return `${widthPercentage}%`;
    },
    getStatName(key) {
      const names = {
        hp: 'HP',
        attack: 'Attack',
        defense: 'Defense',
        'special-attack': 'Spe Atk',
        'special-defense': 'Spe Def',
        speed: 'Speed'
      };
      return names[key] || key;
    },
  },
};
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
  border-radius: 5px; /* Rounded corners for the bar container */
}

.bar {
  height: 100%;
  border-radius: 5px; /* Optionally rounded corners for the bar itself */
  /* Transition for animated width change */
  transition: width 0.5s ease;
}

.stat {
  display: grid;
  grid-template-columns: auto auto 1fr;
  align-items: center;
  gap: 0.5rem;
}
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


