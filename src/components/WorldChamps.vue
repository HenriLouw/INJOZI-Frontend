<template>
  <div class="container-fluid">
    <h1 class="text-center mb-4">F1 World Champions</h1>
    <div v-if="loading" class="text-center my-4">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
      <p class="mt-2">Loading champions...</p>
    </div>
    <div v-else class="row">
      <div class="col-md-6">
        <div class="card border-primary mb-4">
          <div class="card-header bg-primary text-white">Champions List</div>
          <ul class="list-group list-group-flush">
            <li
              v-for="(champion, index) in champions"
              :key="index"
              class="list-group-item champion-item"
              @click="showRaceWinners(champion.year)"
            >
              <span>{{ champion.year }}</span>
              <span>{{ champion.name }}</span>
              <span v-if="champion.isWorldChampion" class="badge bg-warning">World Champion</span>
            </li>
          </ul>
        </div>
      </div>
      <div class="col-md-6">
        <div v-if="selectedYear">
          <h2 class="text-center mb-4">Race Winners {{ selectedYear }}</h2>
          <div class="card border-success">
            <div class="card-header bg-success text-white">Race Winners</div>
            <ul class="list-group list-group-flush">
              <li
                v-for="(winner, index) in raceWinners"
                :key="index"
                class="list-group-item"
              >
                {{ winner }}
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const champions = ref([]);
const selectedYear = ref(null);
const raceWinners = ref([]);
const loading = ref(false);

const fetchChampions = async () => {
  loading.value = true;
  let cachedChampions = localStorage.getItem('f1Champions');
  if (cachedChampions) {
    champions.value = JSON.parse(cachedChampions);
    loading.value = false;
  } else {
    try {
      const response = await fetch('http://ergast.com/api/f1/driverstandings/1.json?limit=100&offset=55');
      const data = await response.json();
      champions.value = data.MRData.StandingsTable.StandingsLists.map((standing) => ({
        year: standing.season,
        name: standing.DriverStandings[0].Driver.givenName + ' ' + standing.DriverStandings[0].Driver.familyName,
        isWorldChampion: standing.DriverStandings[0].position === '1',
      }));
      localStorage.setItem('f1Champions', JSON.stringify(champions.value));
      loading.value = false;
    } catch (error) {
      console.error('Error fetching champions:', error);
      loading.value = false;
    }
  }
};

const fetchRaceWinners = async (year) => {
  try {
    const response = await fetch(`http://ergast.com/api/f1/${year}/results/1.json`);
    const data = await response.json();
    raceWinners.value = data.MRData.RaceTable.Races.map((race) => race.Results[0].Driver.givenName + ' ' + race.Results[0].Driver.familyName);
  } catch (error) {
    console.error('Error fetching race winners:', error);
  }
};

const showRaceWinners = (year) => {
  selectedYear.value = year;
  fetchRaceWinners(year);
};

onMounted(() => {
  fetchChampions();
});
</script>

<style scoped>
.champion-item {
  cursor: pointer;
  transition: background-color 0.3s;
}
.champion-item:hover {
  background-color: #f0f0f0;
}
</style>
