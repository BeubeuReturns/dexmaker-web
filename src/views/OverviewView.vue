<template>
    <div class="overview-container">
      <div class="chart-container">
        <canvas id="typeChart"></canvas>
      </div>
      <div id="chartjs-tooltip" class="custom-tooltip"></div>
    </div>
  </template>
  
  
  <script>
  import Chart from 'chart.js/auto';
  
  export default {
    data() {
      return {
        pokemons: [],
        selectedPokemonIds: [],
    typeColors: {
      'normal': '#e3e3e3',
      'fire': '#e62829',
      'water': '#6390F0',
      'electric': '#F7D02C',
      'grass': '#7AC74C',
      'ice': '#96D9D6',
      'fighting': '#ff8000',
      'poison': '#A33EA1',
      'ground': '#E2BF65',
      'flying': '#A98FF3',
      'psychic': '#F95587',
      'bug': '#A6B91A',
      'rock': '#afa981',
      'ghost': '#735797',
      'dragon': '#6F35FC',
      'dark': '#21130d',
      'steel': '#B7B7CE',
      'fairy': '#D685AD',

    },
  };
},
    
    computed: {
        pokemonsByType() {
    let byType = {};
    this.selectedPokemons.forEach(pokemon => {
      pokemon.types.forEach(type => {
        if (!byType[type]) {
          byType[type] = [];
        }
        byType[type].push(pokemon.name);
      });
    });
    return byType;
  },
      typeCounts() {
        let counts = {};
        this.selectedPokemons.forEach(pokemon => {
          pokemon.types.forEach(type => {
            if (!counts[type]) {
              counts[type] = 0;
            }
            counts[type]++;
          });
        });
        return counts;
      },
      chartData() {
    let types = Object.keys(this.typeCounts);
    let counts = Object.values(this.typeCounts);
    let backgroundColors = types.map(type => this.typeColors[type] || '#999999'); // Fallback color

    return {
      labels: types,
      datasets: [
        {
          label: 'Type Distribution',
          data: counts,
          backgroundColor: backgroundColors,
          hoverOffset: 4
        }
      ]
    };
  },
      selectedPokemons() {
        return this.pokemons.filter(pokemon => this.selectedPokemonIds.includes(pokemon.id));
      }
    },
    mounted() {
      this.selectedPokemonIds = JSON.parse(localStorage.getItem('selectedPokemonIds') || '[]');
      fetch('/dexmaker-web/pokemon_data_sorted.json')
        .then((response) => response.json())
        .then((data) => {
          this.pokemons = data;
          this.renderChart();
        });
    },
    methods: {
        renderChart() {
    var ctx = document.getElementById('typeChart').getContext('2d');
    var customTooltip = document.getElementById('chartjs-tooltip');

    new Chart(ctx, {
      type: 'pie',
      data: this.chartData,
      options: {
        responsive: true,
        interaction: {
          mode: 'point',
        },
        plugins: {
          tooltip: {
            enabled: false, // Disable the default tooltip
            external: (context) => {
              // Tooltip Element
              var tooltipModel = context.tooltip;

              if (tooltipModel.opacity === 0) {
                customTooltip.style.opacity = 0;
                return;
              }

              // Set Text
              if (tooltipModel.body) {
            // Set content with images
            if (tooltipModel.body) {
              var type = tooltipModel.title[0] || '';
              var names = this.pokemonsByType[type] || [];
              customTooltip.innerHTML = ''; // Clear existing content

              // Create and append img elements for each Pokémon
              names.forEach(name => {
                // Assuming the ID is part of your pokemonsByType data structure
                var pokemonId = this.pokemons.find(pokemon => pokemon.name === name).id;
                var imgSrc = `/dexmaker-web/images/pokemons/${pokemonId}.png`;
                var img = new Image(48, 48); // Set image size as needed
                img.src = imgSrc;
                img.alt = name;
                img.title = name.charAt(0).toUpperCase() + name.slice(1); // Capitalize the first letter
                customTooltip.appendChild(img);
              });
            }
              }

              // Display, position, and style the tooltip
              customTooltip.style.opacity = 1;
              customTooltip.style.position = 'absolute';
              customTooltip.style.left = tooltipModel.caretX + 'px';
              customTooltip.style.top = tooltipModel.caretY + 'px';
              customTooltip.style.fontFamily = tooltipModel.options.bodyFont.family;
              customTooltip.style.fontSize = tooltipModel.options.bodyFont.size + 'px';
              customTooltip.style.fontStyle = tooltipModel.options.bodyFont.style;
              customTooltip.style.padding = tooltipModel.options.padding + 'px ' + tooltipModel.options.padding + 'px';
              customTooltip.style.pointerEvents = 'none';
              customTooltip.style.backgroundColor = 'rgba(255, 255, 255, 0.8)';
              customTooltip.style.borderRadius = '6px';
              customTooltip.style.color = 'black';
              customTooltip.style.zIndex = 9999;
              
            }
          },
          legend: {
            position: 'top',
          },
          title: {
            display: true,
            text: 'Pokémon Type Distribution'
          },
        }
      }
    });
  },
    }
  };
  </script>
  
  <style scoped>
  .overview-container {
    display: flex;
    justify-content: space-between; /* Align children (chart and tooltip) on opposite sides */
    align-items: center;
    min-height: 100vh;
    width: 98vw;
    padding: 20px; /* Add some padding around the edges */
  }
  .chart-container {
    width: 50%; /* Adjust the width as needed */
    height: auto; /* Maintain the aspect ratio */
    margin-right: 20px; /* Add some margin between chart and tooltip */
  }
  
  #typeChart {
    width: 100%; /* Make the canvas fill the container */
    height: auto; /* Maintain the aspect ratio */
  }
  
  .custom-tooltip {
    max-width: 400px; /* Adjust the width as needed */
    max-height: 300px;
    overflow-y: auto;
    /* ... other styles ... */
  }
  
  /* Add responsive design if needed */
  @media (max-width: 768px) {
    .overview-container {
      flex-direction: column;
    }
  
    .chart-container {
      width: 100%;
      margin-bottom: 20px;
    }
  
    #typeChart {
      height: auto;
    }
  }
  </style>
  
  