<template>

  <h1>{{currentMode }}</h1>

  <template v-if="currentMode == 'trending'">
    
    <div @click="fetchRandomAnime()" style="position: absolute; right: 0; transform: translateY(-200%);" class="button">Get Random </div>
    <div v-if="fetchedData.length">
      <ul class="list-container">
        <template v-for="(anime, index)  in fetchedData" :key="anime.id" >
          <li data-aos="fade-up" :data-aos-delay="((index % 3) * 100) + 0"  data-aos-duration="1000"  
          >
            <img :src="anime.coverImage.large" :alt="anime.title.romaji" @touchstart="startPress(anime.coverImage.large, $event)" 
          @touchend="cancelPress"
          @touchmove="moveImage($event)"/>

            <p @click="getDetail(anime)">{{ anime.title.native }}</p>
            <div class="star-rating">
                <div class="stars-outer">
                    <div class="stars-inner" :style="{ width: `${anime.averageScore}%` }"></div>
                    
                    
                </div>
                <span>EP:{{ anime.episodes }}</span>
                <!-- <span>EP:999</span> -->
            </div>
            
        </li>
        </template>
      </ul>
    </div>
    <div v-else>
      Loading...
    </div>

    <div v-if="showCircle" 
         :style="{ background: `url(${selectedImage}) center/cover`, 
                   transform: `translate(${imgPosition.x}px, ${imgPosition.y}px)` }" 
         class="circle"
    ></div>
  </template>

  <template v-if="currentMode == 'search'">
    <div class="progress-bar" :style="{ width: progressBarWidth }"></div>
    <!-- <vue-progress-bar v-if="loading"></vue-progress-bar> -->
    <div>
      <input v-model="query" placeholder="Search for Anime...">
    <button @click="getAnime">Search</button>
      <div v-if="loading">Loading...</div>
      <span v-if="searchList"><br><br>Found: {{ searchList.length }}</span>
      <div v-for="anime in searchList" :key="anime.id">
        <!-- {{ anime.data }} -->
        <h3>{{ anime?.title?.native }}</h3>
        <!-- Add more details as needed -->
      </div>
    </div>
  </template>
    

  <div class="radial-menu">
    <div class="pie pie1" :style="getPieStyle('pie1','search')" @click="changeMode('search')">
      <div class="pie-color pie-color1">
        <i class="fas fa-search" style="color: black;"></i>
      </div>
    </div>
    <div class="pie pie2" :style="getPieStyle('pie2','myList')" @click="changeMode('myList')">
      <div class="pie-color pie-color2"></div>
      <i class="fa-solid fa-star"></i>
    </div>
    <div class="pie pie3" :style="getPieStyle('pie3','trending')" @click="changeMode('trending')">
      <div class="pie-color pie-color3"></div>
      <i class="fa-solid fa-fire-flame-curved"></i>
    </div>
    <div class="pie pie4" :style="getPieStyle('pie4','profile')" @click="changeMode('profile')">
      <div class="pie-color pie-color4"></div>
      <i class="fa-solid fa-user"></i>
    </div>
    <div></div>
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
  import axios from 'axios';
  // import VueProgressBar from 'vue-progressbar';

// const options = {
//   color: '#bffaf3',
//   failedColor: '#874b4b',
//   thickness: '5px',
//   // ... other options
// };





  export default {
    name: "TrendingAnime",
    data() {
      return {
        currentMode: 'trending',
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

        showCircle: false,
        selectedImage: 'https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx137908-50af3lKVbst2.jpg',
        pressTimer: null,
        imgPosition: { x: 0, y: 0 },
        lastPosition: { x: 0, y: 0 },

        query: '',
        searchList: null,
        totalResults: 0,
        animeCount: 0,
        mangaCount: 0,

        progressBarWidth: '0%',


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

      getPieStyle(className, mode) {
          let style = {
              transform: `rotate(${this.rotations[className]}deg)`
          };

          if (mode === this.currentMode) {
              style.background = '#FB6350';
          }

          return style;
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


      startPress(image, event) {
        
        this.pressTimer = setTimeout(() => {
          

          this.showCircle = true;
          this.selectedImage = image;
          this.imgPosition = {
            x: event.touches[0].clientX - 50, // Assuming circle radius 50px
            y: event.touches[0].clientY - 50,
          };
        }, 1000); // 1 second
      },
      cancelPress() {
        console.log('touch ends')
        clearTimeout(this.pressTimer);
        this.showCircle = false;

        // Enable vertical scrolling again
        document.body.style.overflowY = 'auto';
      },
      moveImage(event) {
        if (!this.showCircle) return;
        document.body.style.overflowY = 'hidden';

        const currentX = event.touches[0].clientX - 50;
        const currentY = event.touches[0].clientY - 50;
        console.log(currentX,currentY)

        if (Math.abs(currentX - this.lastPosition.x) > 1 || 
            Math.abs(currentY - this.lastPosition.y) > 1) {
          
          // Update the position only if it's more than 5px different
          this.imgPosition = {
            x: currentX,
            y: currentY
          };
          
          // Save the new position
          this.lastPosition = {
            x: currentX,
            y: currentY
          };
        }
      },
      changeMode(newMode){
        console.log('changin mode');
        this.currentMode = newMode
        document.body.classList.toggle('active');
        this.toggleRotation()
        this.searchList = null
      },

      async fetchAPI(data) {
        const api = axios.create({
          baseURL: "https://graphql.anilist.co",
        });

        return api.post("/", data);
      },

      async getAnime() {
        this.loading = true;
        this.startLoading(); // Start the custom progress bar
        // this.$Progress.start();

        const query = `
          query ($id: Int, $page: Int, $perPage: Int, $search: String) {
            Page(page: $page, perPage: $perPage) {
              pageInfo {
                total
                currentPage
                lastPage
                hasNextPage
                perPage
              }
              media(id: $id, search: $search, sort: POPULARITY_DESC) {
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
          }`;

        const variables = {
          search: this.query,
          page: 1,
          perPage: 100,
        };

        let result = null;
        try {
          result = await this.fetchAPI({ query, variables });
          // await this.fetchAPI({ query, variables });
          // const result = await this.fetchAPI({ query, variables });
          this.searchList = result.data.data.Page.media
          // console.log(this.animeList)
          // handle the response
        } catch (err) {
          this.handleErrors(err);
        } finally {
          // this.animeList = result
          // this.$Progress.finish();
          this.loading = false; // This will hide the progress bar
          this.loading = false;
        }
      },

      startLoading() {
      this.loading = true;
      this.progressBarWidth = '0%';
      this.increaseProgress();
    },
    increaseProgress() {
  let width = 0;
  const interval = setInterval(() => {
    width += 5; // Adjust the increment as needed
    this.progressBarWidth = `${width}%`;

    if (width >= 100) {
      clearInterval(interval);
      setTimeout(() => {
        this.loading = false; // Hide the progress bar after a short delay
        this.progressBarWidth = '0%'; // Reset the progress bar width
      }, 500); // Delay to ensure users see the progress bar hit 100%
    }
  }, 100); // Adjust the interval as needed
},
    },
    mounted() {
      console.clear()
      this.fetchTrendingAnime();
      AOS.init();

      this.currentMode = 'search'

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
    overflow-x: hidden;
    overflow-y: auto;
  }

  .button{
    background: #FB6350;
    color: white;
    padding: 5px 15px;
    margin: 0px;
    border-radius: 5px;
    
  }

  .list-container{
    padding-inline-start: unset;

    display: grid;
    grid-template-columns: 31% 31% 31%;
    justify-content: space-between;

    width: 95%;
    margin:auto;

    margin-bottom: 100px;

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

    -webkit-touch-callout: none; /* Disable image context menu on iOS */
    -webkit-user-drag: none;
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
    font-size: 13.5px;
    color: Charcoal;
    margin: 10px 5px 20px 0px;
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

  .star-rating span{
    line-height: 13.5px;
    font-size: 12px;
    position: absolute;
    right: 0;
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
    clip-path: polygon(200px 200px, 0 200px, 0px 116px);

    /* border: 2px solid black; */

    right: -200px;
    bottom: -200px;

    width: 400px;
    /* transform: translateX(200px) translateY(200px); */
    transition: all ease-in-out 400ms;
    /* transition: transform 500s; */

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

    position: relative;
    
  }

  .pie1,.pie3{
    background: white;
    /* border: 2px solid black; */
  }

  .pie2, .pie4 {
    background: 	#E8E8E8;
  }

  .pie i{
    color: black;
    position: absolute;
    left: 50px;
    top: 200px;
    transform: translate(-50%,-175%);
    /* font-size: 10em; */
    
    z-index: 100000;
    font-size: 1.5em;
  }

  .pie2 i{
    transform: translate(-50%, -175%) rotate(-22.5deg) !important;
  }

  .pie3 i{
    transform: translate(-50%, -175%) rotate(-45deg) !important;
  }

  .pie4 i{
    transform: translate(-50%, -175%) rotate(-67.5deg) !important;
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

  .circle {
    width: 125px; /* Circle size */
    height: 125px;
    border-radius: 50%; /* Makes it a circle */
    position: fixed; /* Keeps the circle above other elements */
    z-index: 1000; /* Ensures it's on top */
    touch-action: none; /* Prevents default touch behaviors like scrolling */

    top:0px;
    left: 0px;
    border: 2px black solid;

    transition: all .05s ease-in-out;
  }



  .progress-bar {
    position: fixed;
    top: 0;
    left: 0;
    height: 5px;
    background-color: #10b981; /* Example color */
    transition: width 0.5s ease; /* Smooth transition for width change */
  }
</style>
