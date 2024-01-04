<template>
    <h1>Trending Anime</h1>
    <button @click="fetchRandomAnime()" style="position: absolute; right: 0; transform: translateY(-200%);">Get Random</button>
    <div v-if="fetchedData.length">
      <ul class="list-container">
        <template v-for="(anime, index)  in fetchedData" :key="anime.id" >
          <li data-aos="fade-up" :data-aos-delay="((index % 3) * 100) + 0"  data-aos-duration="1000" @click="getDetail(anime)">
            <img :src="anime.coverImage.large" :alt="anime.title.romaji" />

            <p>{{ anime.title.native }}</p>
            <div class="star-rating">
                <div class="stars-outer">
                    <div class="stars-inner" :style="{ width: `${anime.averageScore}%` }"></div>
                </div>
            </div>
        </li>
        </template>

       

      </ul>
    </div>
    <div v-else>
      Loading...
    </div>

    <div class="radial-menu">
      <div class="pie pie1" :style="getPieStyle('pie1')">
        <div class="pie-color pie-color1"></div>
      </div>
      <div class="pie pie2" :style="getPieStyle('pie2')">
        <div class="pie-color pie-color2"></div>
      </div>
      <div class="pie pie3" :style="getPieStyle('pie3')">
        <div class="pie-color pie-color3"></div>
      </div>
      <div class="pie pie4" :style="getPieStyle('pie4')">
        <div class="pie-color pie-color4"></div>
      </div>
      <div class="menu" onclick="document.body.classList.toggle('active')" @click="toggleRotation()">
        <svg class="hamburger" xmlns="http://www.w3.org/2000/svg" width="100" height="100" viewBox="0 0 100 100">
          <g
            fill="none"
            stroke="#000"
            stroke-width="7.999"
            stroke-linecap="round"
          >
            <path d="M 55,26.000284 L 24.056276,25.999716" />
            <path d="M 24.056276,49.999716 L 75.943724,50.000284" />
            <path d="M 45,73.999716 L 75.943724,74.000284" />
            <path d="M 75.943724,26.000284 L 45,25.999716" />
            <path d="M 24.056276,73.999716 L 55,74.000284" />
          </g>
        </svg>
    </div>
    </div>

</template>

<script>
  import AOS from 'aos'
  import 'aos/dist/aos.css'


  export default {
    name: "TrendingAnime",
    data() {
      return {
        fetchedData: [],
        loading: false,
        testNum: 3,

        ratingPercent: Number,

        isMenuOpen: false,

        rotations: {
          pie1: -90,
          pie2: -67.5,
          pie3: -45,
          pie4: -22.5,
        },
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
              idMal
              title {
                romaji
                english
                native
              }
              type
              endDate {
                year
                month
                day
              }
              startDate {
                year
                month
                day
              }
              studios(isMain: true) {
                nodes {
                  name
                }
              }
              isAdult
              source
              genres
              volumes
              episodes
              chapters
              siteUrl
              status
              averageScore
              meanScore
              popularity
              description
              favourites
              coverImage {
                extraLarge
                medium
                large
                color
              }

                
              }
            }
          }
        `;
        // studios
        // endDate

        const variables = {
          page: 1,
          perPage: 18
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
          this.fetchedData = jsonResponse.data.Page.media;
          // console.table(this.fetchedData)
        } catch (error) {
          console.error('Error fetching data:', error);
        } finally {
          this.loading = false;
        }
      },
      handleErrors(error) {
        console.error('Fetch error:', error);
      },
      async fetchRandomAnime() {
        this.loading = true;
        const totalPages = 500
        const randomPage = Math.floor(Math.random() * totalPages) + 1; // Random page number
        // const randomPage = 50
        console.log(randomPage)
        const query = `
          query ($page: Int, $perPage: Int) {
            Page(page: $page, perPage: $perPage) {
              media(type: ANIME, sort: SCORE_DESC) {
                id
                idMal
                title {
                  romaji
                  english
                  native
                }
                type
                endDate {
                  year
                  month
                  day
                }
                startDate {
                  year
                  month
                  day
                }
                studios(isMain: true) {
                  nodes {
                    name
                  }
                }
                isAdult
                source
                genres
                volumes
                episodes
                chapters
                siteUrl
                status
                averageScore
                meanScore
                popularity
                description
                favourites
                coverImage {
                  extraLarge
                  medium
                  large
                  color
                }
              }
            }
          }
        `;

            const variables = {
              page: randomPage,
              perPage: 15 // Number of items per page
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
              const fetchedData = jsonResponse.data.Page.media;

              // Randomly select entries from the fetched list
              this.fetchedData = fetchedData
              // this.fetchedData = this.selectRandomEntries(fetchedData, 15);
        } catch (error) {
          console.error('Error fetching data:', error);
        } finally {
          this.loading = false;
        }
      },

      getDetail(anime){
        console.log(JSON.stringify(anime, null, 2));
        alert(`
          startDate: ${anime.startDate?.year}/${anime.startDate?.month}/${anime.startDate?.day}\n
          EndDate: ${anime.endDate?.year}/${anime.endDate?.month}/${anime.endDate?.day}\n
          Studios: ${anime.studios?.nodes[0]?.name}\n
          Genres: ${anime.genres}\n
          Status: ${anime.status}\n
          Episodes: ${anime.episodes}\n
          Description: ${anime.description}\n

        `)
        // alert("Object keys: " + Object.keys(anime).join(", "));

      },

      // Helper method to randomly select n entries from an array
      selectRandomEntries(arr, n) {
        let result = new Array(n);
        let len = arr.length;
        let taken = new Array(len);
        if (n > len)
          throw new RangeError("selectRandomEntries: more elements taken than available");
        while (n--) {
          let x = Math.floor(Math.random() * len);
          result[n] = arr[x in taken ? taken[x] : x];
          taken[x] = --len in taken ? taken[len] : len;
        }
        return result;
      },
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
      },

      // menu option -----------
      getPieStyle(className) {
        return {
          transform: `rotate(${this.rotations[className]}deg)`
        };
      },
      toggleRotation() {
        if(!this.isMenuOpen){
          Object.keys(this.rotations).forEach((className) => {
            this.rotations[className] += 90;
          });
          this.isMenuOpen = true
        }else{
          Object.keys(this.rotations).forEach((className) => {
            this.rotations[className] -= 90;
          });
          this.isMenuOpen = false
        }
      },
    },
    mounted() {
      console.clear()
      this.fetchTrendingAnime();
      AOS.init();

      // this.isMenuOpen = true
    },

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

  body{
    position: relative;
  }

  .list-container{
    padding-inline-start: unset;

    display: grid;
    grid-template-columns: 31% 31% 31%;
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
    height: 175px; 
    /* aspect-ratio: 1/1; */
    object-fit: cover; 

    filter: saturate(75%);
  }

  .list-container li p{
    font-weight: bold;
    font-size: .8em;
    margin: 10px 0px 0px;

    line-height: 1.5em;      /* Example line-height */
    height: 3em;             /* Double the line-height for two lines */
    overflow: hidden;        /* Hide any text that overflows */
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    text-overflow: ellipsis; /* Add ellipsis if text overflows */
    white-space: normal;
  }

  .star-rating {
    font-size: 15px;
    color: Charcoal;
    margin: 10px 0px 20px;
    line-height: 1;
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

  /* radial menu ----------------- */

  @import url("https://fonts.googleapis.com/css?family=Crimson+Text&display=swap");
  @import url("https://fonts.googleapis.com/css?family=Work+Sans&display=swap");

  .pie {
    -webkit-tap-highlight-color: transparent;
    background: #000;
    border-radius: 50%;
    /* box-shadow: 0 0 4px 5px rgba(0, 0, 0, 0.2); */
    cursor: pointer;
    height: 400px;
    position: fixed;

    right: 0px;
    bottom: 0px;

    clip-path: polygon(200px 200px, 0 200px, 0px 123.46px);
    clip-path: polygon(200px 200px, 0 200px, 0px 117px);

    right: -200px;
    bottom: -200px;

    width: 400px;
    /* transform: translateX(200px) translateY(200px); */
    transition: transform 400ms;
  }
  .pie-color:hover {
    opacity: 0.85;
  }
  .pie-color:active {
    opacity: 0.7;
  }

  .pie-color {
    width: 100%;
    height: 100%;
    border-radius: 50%;
  }

  .pie-color1,.pie-color3 {
    background: white;
  }

  .pie-color2, .pie-color4 {
    background: #FB6350;
  }

  .menu {
    -webkit-tap-highlight-color: transparent;
    background: #fff;
    border-radius: 50%;
    box-shadow: 0 0 4px 5px rgba(0, 0, 0, 0.2);
    cursor: pointer;
    height: 200px;
    right: -100px;
    bottom: -100px;
    position: fixed;
    width: 200px;
  }

  .hamburger {
    cursor: pointer;
    height: 46px;
    left: 25%;
    position: relative;
    top: 25%;
    width: 46px;

    transform: translate(-25%,-25%);
  }
  .hamburger path {
    transition: transform 300ms;
  }

  .hamburger path:nth-child(1) {
  transform-origin: 25% 29%;
}
.hamburger path:nth-child(2) {
  transform-origin: 50% 50%;
}
.hamburger path:nth-child(3) {
  transform-origin: 75% 72%;
}
.hamburger path:nth-child(4) {
  transform-origin: 75% 29%;
}
.hamburger path:nth-child(5) {
  transform-origin: 25% 72%;
}
.active .pie {
  transform: translateX(0) translateY(0);
}
.active .hamburger path:nth-child(1) {
  transform: rotate(45deg);
}
.active .hamburger path:nth-child(2) {
  transform: scaleX(0);
}
.active .hamburger path:nth-child(3) {
  transform: rotate(45deg);
}
.active .hamburger path:nth-child(4) {
  transform: rotate(-45deg);
}
.active .hamburger path:nth-child(5) {
  transform: rotate(-45deg);
}
</style>
