<template>
  <v-app>
    <v-app-bar color="primary">
      <v-app-bar-title class="text-uppercase">Star Wars Films</v-app-bar-title>
      <v-spacer></v-spacer>
      <v-text-field
        v-model="search"
        label="Search films"
        hide-details
        class="mr-4 mt-3"
      ></v-text-field>
    </v-app-bar>

    <v-main>
      <v-container>
        <v-row v-if="loading">
          <v-col cols="12" class="text-center">
            <v-progress-circular indeterminate color="primary"></v-progress-circular>
          </v-col>
        </v-row>
        <v-row v-else-if="error">
          <v-col cols="12">
            <v-alert type="error">{{ error }}</v-alert>
          </v-col>
        </v-row>
        <v-row v-else>
          <v-col v-for="film in filteredFilms" :key="film.episode_id" cols="12" sm="6" md="4" lg="3">
            <v-card class="film-card">
              <v-card-title class="text-h6">{{ film.title }}</v-card-title>
              <v-card-text>
                <p><strong>Released:</strong> {{ film.release_date }}</p>
                <p><strong>Director:</strong> {{ film.director }}</p>
                <p><strong>Producer:</strong> {{ film.producer }}</p>
                <v-expansion-panels class="mt-3">
                  <v-expansion-panel>
                    <v-expansion-panel-title>Opening Crawl</v-expansion-panel-title>
                    <v-expansion-panel-text>{{ film.opening_crawl }}</v-expansion-panel-text>
                  </v-expansion-panel>
                </v-expansion-panels>
                <v-btn @click="openCharacterModal(film)" color="info" class="mt-4" block>
                  View Characters
                </v-btn>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <v-dialog v-model="charactersDialog" max-width="600px">
      <v-card>
        <v-card-title class="text-h5 grey lighten-2">
          Characters - {{ selectedFilm?.title }}
          <v-spacer></v-spacer>
          <v-btn @click="closeCharactersModal">
            Close
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-list v-if="!characterLoading && characters.length">
            <v-list-item v-for="character in characters" :key="character.url">
              <v-list-item-title>{{ character.name }}</v-list-item-title>
              <v-list-item-subtitle>
                <strong>Height:</strong> {{ character.height }}, 
                <strong>Mass:</strong> {{ character.mass }},
                <strong>Hair:</strong> {{ character.hair_color }}, 
                <strong>Skin:</strong> {{ character.skin_color }}, 
                <strong>Eyes:</strong> {{ character.eye_color }},
                <strong>Birth Year:</strong> {{ character.birth_year }}
              </v-list-item-subtitle>
            </v-list-item>
          </v-list>
          <div v-else-if="characterLoading" class="text-center pa-4">
            <v-progress-circular indeterminate color="primary"></v-progress-circular>
          </div>
          <p v-else class="text-center">No characters found.</p>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-app>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'

const API_BASE_URL = 'https://swapi.dev/api'

export default {
  name: 'StarWarsFilmsCatalog',
  
  setup() {
    const films = ref([])
    const search = ref('')
    const loading = ref(true)
    const error = ref(null)
    const charactersDialog = ref(false)
    const selectedFilm = ref(null)
    const characters = ref([])
    const characterLoading = ref(false)

    const filteredFilms = computed(() => {
      const searchFor = search.value.toLowerCase()
      return films.value.filter(film => 
        film.title.toLowerCase().includes(searchFor) ||
        film.director.toLowerCase().includes(searchFor)
      )
    })

    const fetchFilms = async () => {
      try {
        const response = await axios.get(`${API_BASE_URL}/films/`)
        films.value = response.data.results.sort((a, b) => a.episode_id - b.episode_id)
        loading.value = false
      } catch (err) {
        error.value = 'Failed to fetch films. Please try again later'
        loading.value = false
      }
    }

    const openCharacterModal = async (film) => {
      selectedFilm.value = film
      charactersDialog.value = true
      characters.value = []
      characterLoading.value = true

      try {
        const characterPromises = film.characters.map(url => axios.get(url))
        const characterResponses = await Promise.all(characterPromises)
        characters.value = characterResponses.map(response => response.data)
      } catch (err) {
        error.value = 'Failed to fetch characters. Please try again later'
      } finally {
        characterLoading.value = false
      }
    }

    const closeCharactersModal = () => {
      charactersDialog.value = false
      selectedFilm.value = null
      characters.value = []
    }

    onMounted(fetchFilms)

    return {
      films,
      search,
      loading,
      error,
      charactersDialog,
      selectedFilm,
      characters,
      characterLoading,
      filteredFilms,
      openCharacterModal,
      closeCharactersModal
    }
  },

}
</script>