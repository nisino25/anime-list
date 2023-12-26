<template>
  <div>
    <h1>Trending Anime</h1>
    <div v-if="trendingAnime.length">
      <ul class="list-container">
        <li v-for="anime in trendingAnime" :key="anime.id">
          <img :src="anime.coverImage.large" :alt="anime.title.romaji" />
          <!-- <h2>{{ anime.title.romaji }}</h2> -->

          <p>{{ anime.title.native }}</p>
          <!-- <span v-for="n in 5" :key="n" class="star" :class="{ 'filled': n <= computedStars(anime.averageScore) }">★</span> -->
          <!-- <span v-for="n in 5" :key="n" class="star" :class="starClass(n)">★</span> -->
          <!-- <span v-for="n in 5" :key="n" class="star" :class="starClass(n, anime.averageScore)">★</span> -->
          <div class="star-rating">
            <div class="stars-outer">
              <div class="stars-inner" :style="{ width: `${anime.averageScore}%` }"></div>
            </div>
          </div>
          
          <!-- <p>Score: {{ anime.averageScore }}</p> -->
        </li>
      </ul>
    </div>
    <div v-else>
      Loading...
    </div>
  </div>
</template>
<script>
export default {
  name: "TrendingAnime",
  data() {
    return {
      trendingAnime: [],
      loading: false,
      testNum: 3,

      ratingPercent: Number,
    };
  },
  methods: {
    async fetchTrendingAnime() {
      this.loading = true;
      const query = `
        query ($page: Int, $perPage: Int) {
          Page(page: $page, perPage: $perPage) {
            media(type: ANIME, sort: TRENDING_DESC) {
              id
              title {
                native
                romaji
                english
              }
              averageScore
              coverImage {
                large
              }
            }
          }
        }
      `;

      const variables = {
        page: 1,
        perPage: 12
      };

      try {
        const response = await fetch('https://graphql.anilist.co', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
          },
          body: JSON.stringify({
            query,
            variables
          })
        });
        const jsonResponse = await response.json();
        this.trendingAnime = jsonResponse.data.Page.media;
      } catch (error) {
        console.error('Error fetching data:', error);
      } finally {
        this.loading = false;
      }
    },
    handleErrors(error) {
      console.error('Fetch error:', error);
    },
    // starClass(starNumber) {
    //   const fillLevel = Math.ceil((this.ratingPercent / 100) * 5);
    //   if (starNumber <= fillLevel) {
    //     return 'filled';
    //   } else if (starNumber === fillLevel + 1) {
    //     // Check for 25%, 50%, and 75% fill
    //     const partialFill = this.ratingPercent % 20;
    //     if (partialFill >= 15) return 'three-quarter-filled';
    //     if (partialFill >= 10) return 'half-filled';
    //     if (partialFill >= 5) return 'quarter-filled';
    //   }
    //   return '';
    // },
    starClass(starNumber, score) {
      const fillLevel = Math.ceil((score / 100) * 5);
      if (starNumber <= fillLevel) {
        // console.log(';kkk')
        return 'filled';
      } else if (starNumber === fillLevel + 1) {
        // Check for 25%, 50%, and 75% fill
        const partialFill = this.ratingPercent % 20;
        console.log(partialFill)
        if (partialFill >= 15) return 'three-quarter-filled';
        if (partialFill >= 10) return 'half-filled';
        if (partialFill >= 5) return 'quarter-filled';
      }
    }
  },
  mounted() {
    this.fetchTrendingAnime();
  },
  // props: {
  //   ratingPercent: Number
  // },

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

  .list-container{
    padding-inline-start: unset;

    display: grid;
    grid-template-columns: 31.5% 31.5% 31.5%;
    justify-content: space-between;

    width: 95%;
    margin:auto;

    /* background: red; */
  }

  .list-container li{
    list-style: none;
  }

  .list-container li img{
    display: block;
    margin: auto;
    /* max-width: 100%; */
    border: 2px solid black;
    border-radius: 10px;

    max-width: 100%; /* Set the maximum width to prevent stretching */
    height: 175px; /* Set the fixed height to 155px */
    object-fit: cover; 
  }

  .list-container li p{
    font-weight: bold;
    margin: 10px 0px;
  }

.star-rating {
  font-size: 15px;
  color: Charcoal;
  margin: 10px 0px;
}

.stars-outer {
  display: inline-block;
  position: relative;
  unicode-bidi: bidi-override;
}

.stars-outer::before {
  content: '★★★★★';
}

.stars-inner {
  position: absolute;
  top: 0;
  left: 0;
  white-space: nowrap;
  overflow: hidden;
  width: 0;
  color: #FB6350;
}

.stars-inner::before {
  content: '★★★★★';
}
</style>
