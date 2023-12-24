<template>
  <div>
    <h1>Popular Anime Rankings</h1>
    <div v-if="loading">Loading...</div>
    <div v-else>
      <ul>
        <li v-for="(anime, index) in animes" :key="anime.id">
          <h3>{{ index + 1 }}. {{ anime.title.romaji }}</h3>
          <img :src="anime.coverImage.medium" :alt="anime.title.romaji" />
          <p>Total Episodes: {{ anime.episodes }}</p>
          <p>Aired: {{ anime.seasonYear }}</p>
          <p>Popularity: {{ anime.popularity }}</p>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      animes: [],
      loading: false,
      error: null
    };
  },
  methods: {
    async fetchPopularAnime() {
      this.loading = true;
      const query = `
        {
          Page(page: 1, perPage: 10) {
            media(sort: POPULARITY_DESC, type: ANIME) {
              id
              title {
                romaji
              }
              coverImage {
                medium
              }
              episodes
              seasonYear
              popularity
            }
          }
        }
      `;
      try {
        const response = await fetch('https://graphql.anilist.co', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
          },
          body: JSON.stringify({ query })
        });
        const jsonResponse = await response.json();
        this.animes = jsonResponse.data.Page.media;
      } catch (error) {
        console.error('Fetch error:', error);
        this.error = error;
      } finally {
        this.loading = false;
      }
    }
  },
  mounted() {
    this.fetchPopularAnime();
  }
};
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
