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
    <div v-if="!isProgressMax" class="progress-bar" :style="{ width: progressBarWidth }"></div>
    <!-- <vue-progress-bar v-if="loading"></vue-progress-bar> -->
    <div>
      <input v-model="query" placeholder="Search for Anime...">
    <button @click="searchAnime">Search</button>
      <div v-if="loading">Loading...</div>
      <span v-if="searchList"><br><br>Found: {{ searchList.length }}</span>
      

      <ul class="binary-list-container">
        <template v-for="(anime, index)  in searchList" :key="anime.id" >
          <li data-aos="fade-up" :data-aos-delay="((index % 3) * 100) + 0"  data-aos-duration="1000"  
          >
          <div class="image-container">
            <img :src="anime.coverImage.large" :alt="anime.title.romaji" @touchstart="startPress(anime.coverImage.large, $event)" 
            @touchend="cancelPress"
            @touchmove="moveImage($event)"/>
          </div>
            
            <div class="list-bottom">
              <p @click="getDetail(anime)">{{ anime.title.native }}</p>
              <span class="badge" :style="getBadgeStyle(anime.format)">{{anime.format}}</span>
              <span v-html="fetchContentMetrics(anime)"></span><br>

              <template v-for="(genre,index)  in anime.genres" :key="index" >
                <span v-if="index<2" class="badge" :style="getBadgeStyle(genre)">{{genre}}</span>
              </template>

              <span class="duration" v-if="anime.format !== 'MOVIE'" ><i class="fa-regular fa-calendar"></i>  {{ anime.startDate?.year }} - {{ anime.endDate?.year }}</span>
              <span class="duration" v-else ><i class="fa-regular fa-calendar"></i>  {{ anime.startDate?.year }}</span>

              <div class="star-rating">
                <div class="stars-outer">
                  <div class="stars-inner" :style="{ width: `${anime.averageScore}%` }"></div>
                </div>
                <div class="favorite-counter"><i class="fa-solid fa-star"></i>{{ getAnimeFavourites(anime.favourites) }}</div>
                
              </div>
              
              

              <span class="description" v-html="anime.description" @click="anime.showingDetail = !anime.showingDetail" :class="{ showingDetail: anime.showingDetail }"></span>
                
              </div>
              
            
          </li>
        </template>
      </ul>
      <!-- <div v-for="anime in searchList" :key="anime.id">
        <h3>{{ anime?.title?.native }}</h3>
      </div> -->
    </div>
  </template>

  <template v-if="currentMode == 'popular'">
    <div v-if="!isProgressMax" class="progress-bar" :style="{ width: progressBarWidth }"></div>
    <ul class="single-row-list-container">
        <template v-for="(anime, index)  in fetchedData" :key="anime.id" >
          <li>
            <div class="image-container">
              <img :src="anime.coverImage.medium" :alt="anime.title.romaji" @touchstart="startPress(anime.coverImage.large, $event)" 
              @touchend="cancelPress"
              @touchmove="moveImage($event)"/>
            </div>
            No.{{ index+1 }}. {{ anime.title.native }}. <span style="float: right;">{{ getAnimeFavourites(anime.favourites) }}</span>
          </li>
        </template>
    </ul>
    <div @click="fetchPopularAnime()" style="position: absolute; left: 0; transform: translateY(0%);margin-bottom: 50px; display: block;" class="button">Get More </div>
  </template>
    

  <div class="radial-menu">
    <div class="pie pie1" :style="getPieStyle('pie1','search')" @click="changeMode('search')">
      <div class="pie-color pie-color1">
        <i class="fas fa-search" style="color: black;"></i>
      </div>
    </div>
    <div class="pie pie2" :style="getPieStyle('pie2','popular')" @click="changeMode('popular')">
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
    <div class="menu" @click="toggleRotation()">
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
        isProgressMax: false,

        popularPage: 1,


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
          document.body.classList.toggle('active');
        }else{
          Object.keys(this.rotations).forEach((className) => {
            this.rotations[className] -= 90;
          });
          this.isMenuOpen = false
          document.body.classList.toggle('active');
        }
      },
      async changeMode(newMode){
        this.currentMode = newMode;
        this.fetchedData = [];
        
        setTimeout(() => {
              this.toggleRotation()
              
            }, 750); // Delay to ensure users see the progress bar hit 100%
        

        if(newMode == 'search') this.searchList = null
        if(newMode == 'trending') this.fetchTrendingAnime()
        if(newMode == 'popular') this.fetchPopularAnime()
        
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
      

      async searchAnime() {
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
                format
                duration
                volumes
              }
            }
          }`;

        const variables = {
          search: this.query,
          page: 1,
          perPage: 100,
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
          // console.log(jsonResponse)
          // this.fetchedData = jsonResponse.data.Page.media;
          this.searchList = jsonResponse.data.Page.media
          console.log(this.searchList)
          // console.table(this.fetchedData)
        } catch (error) {
          console.error('Error fetching data:', error);
        } finally {
          this.loading = false;
        }
      },

      startLoading() {
        this.loading = true;
        this.isProgressMax = false
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
              this.isProgressMax = true
            }, 750); // Delay to ensure users see the progress bar hit 100%
            
          }
        }, 100); // Adjust the interval as needed
      },

      getBadgeStyle(format){
        let style = `background: `
        if(format == 'TV') {
          return style+'#66CC66;'
        }else if(format == 'MOVIE') {
          return style+'#99C0CC;'
        }else if(format == 'MANGA')
        {
          return style+'#B39EB5;'
        }else{
          return style+'#A0A0A0;'
        }
      },

      fetchContentMetrics(anime){
        const format = anime.format
        if(format == 'TV') return `EP: ${anime.episodes}`
        if(format == 'MOVIE') {
          if (anime.duration < 60) {
              return "<i class='fa-regular fa-clock'></i> " + anime.duration + " min";
          } else {
              let hours = Math.floor(anime.duration / 60);
              let remainingMinutes = anime.duration % 60;
              return "<i class='fa-regular fa-clock'></i> " + hours + "h " + remainingMinutes + "m";
          }
        }
        if(format == 'MANGA') return `Vol: ${anime.volumes}`
      },

      getAnimeFavourites(num) {
          // If the number is less than 100, return it as it is
          if (num < 100) {
              return num;
          }
          // If the number is 100 or more, divide by 1000 and append 'k'
          else {
              return (num / 1000).toFixed(1) + 'k';
          }
      },

      async fetchPopularAnime() {
        this.loading = true;
        this.startLoading(); // Start the custom progress bar
        // this.$Progress.start();
        const query = `
          query ($page: Int, $perPage: Int) {
            Page(page: $page, perPage: $perPage) {
              media(type: ANIME, sort: POPULARITY_DESC) {
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
          page: this.popularPage,
          perPage: 18
        };

        console.log(this.popularPage)
        this.popularPage++

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
          // this.fetchedData = jsonResponse.data.Page.media;
          this.fetchedData = this.fetchedData.concat(jsonResponse.data.Page.media);

          console.log(this.fetchedData)
        } catch (error) {
          console.error('Error fetching data:', error);
        } finally {
          this.loading = false;
        }
      },
    },
      mounted() {
        console.clear()
        // this.fetchTrendingAnime();
        AOS.init();

        this.currentMode = 'popular'
        this.query = 'my hero'
        this.fetchPopularAnime()

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
    position: relative;
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
    transition: all ease-in-out 750ms;
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
    background-color: #FB6350; /* Example color */

    transition: width 0.5s ease; /* Smooth transition for width change */
  }


  .binary-list-container{
    margin: 0;
    padding: 0;

    padding-inline-start: unset;
    display: grid;
    grid-template-columns: 48% 48%;
    justify-content: space-between;
    width: 95%;
    margin: auto;
    margin-bottom: 100px 
  }

  

  .binary-list-container li {
    list-style: none;
    border: 2px solid DimGray;
    margin-bottom: 10px;
    border-radius: 8px;
    background: #E6D4D0;
    /* box-shadow: #000; */
    box-shadow: rgba(149, 157, 165, 0.5) 0px 8px 24px;

    overflow: hidden;

    transition: all 2s ease; 



  }

  .binary-list-container li .image-container{
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    aspect-ratio: 1/1;
    overflow: hidden; 
    position: relative;
    /* border-bottom: 2px solid DimGray; */
  }

  .binary-list-container li .image-container img{
    position: absolute;
    top: 50%;
    left:50%;
    transform: translate(-50%,-50%);
    /* max-width: 100%;
    max-height: 100%; */
    /* width: 100%;
    aspect-ratio: 5/4;
    overflow: hidden; Ensures the image stays within the bounds of the container */
  }

  .binary-list-container li .list-bottom{
    padding: 10px 7.5px;
  }

  .binary-list-container li .list-bottom p{
    margin: 0 0 2.5px;
    line-height: 1.5em;      /* Example line-height */
    height: 3em;             /* Double the line-height for two lines */
    overflow: hidden;        /* Hide any text that overflows */
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    text-overflow: ellipsis; /* Add ellipsis if text overflows */
    white-space: normal;
  }

  .binary-list-container li .list-bottom .badge {
      padding: 3px 12.5px;
      border-radius: 5px;
      color: white;
      font-weight: bold;
      text-align: center;
      font-size: 12.5px;
      display: inline-block;
      margin: 5px 10px 5px 0px;
      /* float: right; */
  }

  .binary-list-container li .list-bottom .duration{
    margin-top: 7.5px;
    display: block;
  }

  /* .binary-list-container li .list-bottom .star-rating{
    margin-bottom: 5px;
  } */

  .binary-list-container li .list-bottom .description{
    transition: all 2s ease; 
    margin: 0;
    line-height: 1.5em;      /* Example line-height */
    max-height: 4.5em;             /* Double the line-height for two lines */
    overflow: hidden;        /* Hide any text that overflows */
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    text-overflow: ellipsis; /* Add ellipsis if text overflows */
    white-space: normal;
  }

  .binary-list-container li .list-bottom .showingDetail{
    max-height: 150em; /* Set it to 'auto' to smoothly expand */
    overflow: hidden;
    -webkit-line-clamp: initial; /* Reset the line clamp */
    -webkit-box-orient: initial; /* Reset the box orientation */
    text-overflow: initial; /* Reset text overflow */
    white-space: initial; /* Reset white space */
  }

  .favorite-counter{
    position: absolute;
    top: 0px;
    right: 0;
    font-weight: bold;
    display: flex;
    align-items: center;
  }

  .favorite-counter i {
    color: 	#FF8C00;
    margin-right: 2.5px;
    transform: translateY(-1.5px);
  }


  .single-row-list-container{
    margin: 0;
    margin-bottom: 50px;
    padding-left: 0;
    list-style: none;
  }

  .single-row-list-container li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border: 2px solid black;
    margin: 10px auto;
  }

  .single-row-list-container .image-container{

  }

  .single-row-list-container li img{
    height: 100px;
    margin: 0;
    margin-right: 10px;

    display: block; /* Set the image to display as a block element */
    vertical-align: bottom;
  }

</style>
