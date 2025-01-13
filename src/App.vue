<template>
  <div class="min-h-screen bg-spotify-black text-spotify-white p-8">
    <h1 class="text-4xl font-bold mb-8">Song Recommender</h1>
    
    <!-- Song List -->
    <div class="bg-spotify-dark-gray rounded-lg p-6 mb-8">
      <h2 class="text-2xl font-semibold mb-4">Select Songs</h2>
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        <div v-for="song in paginatedSongs" :key="`${song.artist_name}-${song.track_name}`" 
             class="flex items-center space-x-4 p-4 rounded-md hover:bg-spotify-light-gray transition duration-300">
          <input type="checkbox" 
                 :id="`song-${song.artist_name}-${song.track_name}`"
                 v-model="selectedSongs" 
                 :value="song"
                 class="form-checkbox h-5 w-5 text-spotify-green rounded focus:ring-spotify-green focus:ring-offset-spotify-black">
          <label :for="`song-${song.artist_name}-${song.track_name}`" class="flex-grow cursor-pointer">
            <div class="font-medium">{{ song.track_name }}</div>
            <div class="text-spotify-light-gray text-sm">{{ song.artist_name }}</div>
          </label>
        </div>
      </div>
      
      <!-- Pagination -->
      <div class="flex justify-between items-center mt-6">
        <button @click="prevPage" 
                :disabled="currentPage === 1" 
                class="px-4 py-2 rounded-full bg-spotify-green text-spotify-black font-medium disabled:opacity-50 disabled:cursor-not-allowed">
          <ChevronLeftIcon class="h-5 w-5" />
        </button>
        <span>Page {{ currentPage }} of {{ totalPages }}</span>
        <button @click="nextPage" 
                :disabled="currentPage === totalPages" 
                class="px-4 py-2 rounded-full bg-spotify-green text-spotify-black font-medium disabled:opacity-50 disabled:cursor-not-allowed">
          <ChevronRightIcon class="h-5 w-5" />
        </button>
      </div>
    </div>
    
    <!-- Get Recommendations Button -->
    <button @click="getRecommendations" 
            :disabled="selectedSongs.length === 0"
            class="w-full py-3 rounded-full bg-spotify-green text-spotify-black font-bold text-lg hover:bg-spotify-bright-green transition duration-300 disabled:opacity-50 disabled:cursor-not-allowed">
      Get Recommendations
    </button>
    
    <div class="space" style="margin: 2em;"></div>

    <!-- Update Model Button -->
    <button @click="updateModel" 
            class="w-full py-3 rounded-full bg-spotify-light-gray text-spotify-black font-bold text-lg hover:bg-spotify-bright-gray transition duration-300 disabled:opacity-50 disabled:cursor-not-allowed">
      Update Model
    </button>
    
    <!-- Recommendations List -->
    <div v-if="recommendations.length > 0" class="mt-8 bg-spotify-dark-gray rounded-lg p-6">
      <h2 class="text-2xl font-semibold mb-4">Recommended Songs</h2>
      <ul class="space-y-2">
        <li v-for="song in recommendations" :key="`${song.artist_name}-${song.track_name}`" 
            class="flex items-center space-x-4 p-4 rounded-md hover:bg-spotify-light-gray transition duration-300">
          <MusicIcon class="h-6 w-6 text-spotify-green" />
          <div>
            <div class="font-medium">{{ song.track_name }}</div>
            <div class="text-spotify-light-gray text-sm">{{ song.artist_name }}</div>
          </div>
        </li>
      </ul>
    </div>
  </div>

  <!-- Rodapé exibindo model_date e model_version -->
  <footer class="mt-8 p-4 bg-spotify-dark-gray text-spotify-light-gray rounded">
    <p>Model Date: {{ model_date }}</p>
    <p>Model Version: {{ model_version }}</p>
  </footer>

</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { ChevronLeftIcon, ChevronRightIcon, MusicIcon } from 'lucide-vue-next'


const songs = ref([])

const staticSongs = [
  { artist_name: "Lynyrd Skynyrd", track_name: "Sweet Home Alabama"},
  { artist_name: 'Queen', track_name: 'Bohemian Rhapsody' },
  { artist_name: "Guns N' Roses", track_name: "Sweet Child O' Mine" },
  { artist_name: 'Eagles', track_name: 'Hotel California - Remastered' },
  { artist_name: 'Nirvana', track_name: 'Smells Like Teen Spirit' },
  { artist_name: 'The Eagles', track_name: 'Hotel California' },
  { artist_name: 'The Rolling Stones', track_name: 'Satisfaction' }
]

const selectedSongs = ref([])
const recommendations = ref([])
const model_date = ref('')
const model_version = ref('')

// Pagination
const currentPage = ref(1)
const songsPerPage = 6

const totalPages = computed(() => Math.ceil(songs.value.length / songsPerPage))

const paginatedSongs = computed(() => {
  const start = (currentPage.value - 1) * songsPerPage
  const end = start + songsPerPage
  return songs.value.slice(start, end)
})

const prevPage = () => {
  if (currentPage.value > 1) currentPage.value--
}

const nextPage = () => {
  if (currentPage.value < totalPages.value) currentPage.value++
}

onMounted(() => {getSongs()})

// Get recommendations (replace with actual API call)
const getRecommendations = async () => {
  const response = await fetch(`${process.env.VUE_APP_API_URL}/recommend`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(selectedSongs.value)
  })
  const data = await response.json()
  model_date.value = formatDateString(data.model_date)
  model_version.value = data.model_version
  recommendations.value = data.recommendations || []
  console.log(data)
}

const updateModel = async () => {
  const body = {
    dataset_id: (Number(model_version) % 2 == 0) ? "2023_spotify_ds1.csv" : "2023_spotify_ds2.csv"
  }
  console.log(body)
  const response = await fetch(`${process.env.VUE_APP_API_URL}/update-model`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body)
  })
  const data = await response.json()
  model_date.value = formatDateString(data.model_date)
  model_version.value = data.model_version
  recommendations.value = []
}

// Get Songs
const getSongs = async () => {
  const response = await fetch(`${process.env.VUE_APP_API_URL}/tracks`)
  const data = await response.json()
  songs.value = [...staticSongs, ...data.songs]
  model_date.value = formatDateString(data.model_date)
  model_version.value = data.model_version
}

function formatDateString(str) {
  // "2025-01-13 15:24:24.819387" => "2025-01-13" e "15:24:24.819387"
  const [datePart, timePart] = str.split(' ');

  // datePart => "2025-01-13"
  const [year, month, day] = datePart.split('-');

  // timePart => "15:24:24.819387"
  // se não ligar para milissegundos, pode pegar só os 3 primeiros segmentos
  const [h, m, sMillis] = timePart.split(':');
  const s = sMillis.split('.')[0]; // "24" antes do "."

  // Monta no formato "dd/mm/yyyy HH:mm:ss"
  return `${day}/${month}/${year} ${h}:${m}:${s}`;
}
</script>

<style>
:root {
  --spotify-black: #191414;
  --spotify-dark-gray: #282828;
  --spotify-light-gray: #b3b3b3;
  --spotify-white: #ffffff;
  --spotify-green: #1DB954;
  --spotify-bright-green: #1ed760;
}

.bg-spotify-black { background-color: var(--spotify-black); }
.bg-spotify-dark-gray { background-color: var(--spotify-dark-gray); }
.bg-spotify-light-gray { background-color: var(--spotify-light-gray); }
.text-spotify-white { color: var(--spotify-white); }
.text-spotify-light-gray { color: var(--spotify-light-gray); }
.text-spotify-black { color: var(--spotify-black); }
.bg-spotify-green { background-color: var(--spotify-green); }
.text-spotify-green { color: var(--spotify-green); }
.hover\:bg-spotify-light-gray:hover { background-color: var(--spotify-light-gray); }
.hover\:bg-spotify-bright-green:hover { background-color: var(--spotify-bright-green); }
.focus\:ring-spotify-green:focus { --tw-ring-color: var(--spotify-green); }
.focus\:ring-offset-spotify-black:focus { --tw-ring-offset-color: var(--spotify-black); }
</style>