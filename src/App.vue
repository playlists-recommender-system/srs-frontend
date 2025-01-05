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
</template>

<script setup>
import { ref, computed } from 'vue'
import { ChevronLeftIcon, ChevronRightIcon, MusicIcon } from 'lucide-vue-next'

// Sample song list (replace with your actual song list or API call)
const songs = ref([
  { artist_name: 'The Beatles', track_name: 'Hey Jude' },
  { artist_name: 'Queen', track_name: 'Bohemian Rhapsody' },
  { artist_name: 'Led Zeppelin', track_name: 'Stairway to Heaven' },
  { artist_name: 'Bob Dylan', track_name: 'Like a Rolling Stone' },
  { artist_name: 'The Rolling Stones', track_name: 'Satisfaction' },
  { artist_name: 'Jimi Hendrix', track_name: 'Purple Haze' },
  { artist_name: 'U2', track_name: 'With or Without You' },
  { artist_name: 'The Eagles', track_name: 'Hotel California' },
  { artist_name: 'Nirvana', track_name: 'Smells Like Teen Spirit' },
  { artist_name: 'David Bowie', track_name: 'Space Oddity' },
  { artist_name: "Lynyrd Skynyrd", track_name: "Sweet Home Alabama"}
  // Add more songs as needed
])

const selectedSongs = ref([])
const recommendations = ref([])

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

// Get recommendations (replace with actual API call)
const getRecommendations = async () => {
  // Simulating an API call
  console.log('Selected songs:', selectedSongs.value)
  
  // In a real application, you would make an API call here
  const response = await fetch('http://127.0.0.1:8000/recommend', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(selectedSongs.value)
  })
  const data = await response.json()
  recommendations.value = data.recommendations
  console.log(recommendations.value)
  // For this example, we'll just return a subset of the original songs
  // recommendations.value = songs.value
  // .filter(song => !selectedSongs.value.includes(song))
  // .sort(() => 0.5 - Math.random())
  // .slice(0, 5)
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