<template>
  <div class="container-fluid p-4">
    <h1 class="text-center mb-4">F1 World Champions</h1>
    <div v-if="loading" class="text-center my-4 ">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
      <p class="mt-2">Loading champions...</p>
    </div>
    <div v-else class="row  d-flex justify-content-center">
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
              <span v-if="champion.isWorldChampion" class="badge bg-warning me-2">World Champion</span>
              <span class="fw-bold">{{ champion.year }}</span>
              <span class="ms-2">{{ champion.name }}</span>
            </li>
          </ul>
        </div>
      </div>
      <div v-if="selectedYear" class="row">
        <div class="col-md-12 text-center mb-4">
          <h1>Race Winners {{ selectedYear }}</h1>
        </div>
        <div class="row d-flex justify-content-center">
          <div class="col-md-6 ">
            <div class="card border-success">
              <div class="card-header bg-success text-white">Race Winners</div>
              <ul class="list-group list-group-flush">
                <li
                  v-for="(winner, index) in raceWinners"
                  :key="index"
                  :class="{ 'bg-success text-white': isWorldChampion(winner) }"
                  class="list-group-item"
                >
                  <span class="trophy-icon me-2">🏆</span>
                  <span class="fw-bold">{{ winner }}</span>
                </li>
              </ul>
            </div>
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

const isWorldChampion = (winner) => {
  return champions.value.some(champion => winner.includes(champion.name) && selectedYear.value === champion.year);
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
.trophy-icon {
  font-size: 1.2rem;
}
</style>
