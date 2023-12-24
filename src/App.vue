<template>
    <div>
    <h1>Anime Details</h1>
    <div v-if="loading">Loading...</div>
    <div v-else-if="animeData">
      <h2>{{ animeData.title.romaji }}</h2>
      <p>English Title: {{ animeData.title.english }}</p>
      <p>Native Title: {{ animeData.title.native }}</p>
    </div>
  </div>
</template>

<script>
  // import HelloWorld from './components/HelloWorld.vue'

  export default {
    name: 'App',
    
    data() {
      return {
        animeData: null,
        loading: false,
        error: null
      };
    },
    methods: {
      async fetchAnimeData() {
        this.loading = true;
        const query = `
          query ($id: Int) {
            Media (id: $id, type: ANIME) {
              id
              title {
                romaji
                english
                native
              }
            }
          }
        `;
        const variables = { id: 15125 }; // You can change the ID to fetch different anime
        try {
          const response = await fetch('https://graphql.anilist.co', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
              'Accept': 'application/json',
            },
            body: JSON.stringify({ query, variables })
          });
          const jsonResponse = await response.json();
          if (jsonResponse.errors) {
            throw new Error('Error fetching data');
          }
          this.animeData = jsonResponse.data.Media;
        } catch (error) {
          console.error('Fetch error:', error);
          this.error = error;
        } finally {
          this.loading = false;
        }
      }
    },
    mounted() {
      this.fetchAnimeData();
    }
  }
</script>

<style>
  /* #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  } */

  html{
    background: #F7EADF;
    font-family: 'Roboto Condensed', sans-serif;
    /* #FB6350 */
  }
</style>
