<template>

  <h1>{{currentMode }}</h1>
  <div v-if="!isProgressMax" class="progress-bar" :style="{ width: progressBarWidth }"></div>


  <div v-if="currentMode == 'profile'" @click="resetLocaldata()" style="position: absolute; right: 0; top:10px; transform: translateY(0%); display: block;" class="button">Rest localData</div>

  <div v-if="myAnimeListTab !== 'past'" class="tab-wrapper" style="grid-template-columns: 22% 22% 22% 22%;">
      <div @click="listStyle = 'detail'" class="tab"  :style="{ background: listStyle == 'detail' ? '#FB6350' : '' }">
        <span>detail</span>
      </div>
      <div @click="listStyle = 'single'" class="tab"  :style="{ background: listStyle == 'single' ? '#FB6350' : '' }">
        <span>single</span>
      </div>
      <div @click="listStyle = 'double'" class="tab"  :style="{ background: listStyle == 'double' ? '#FB6350' : '' }">
        <span>double</span>
      </div>
      <div @click="listStyle = 'triple'" class="tab"  :style="{ background: listStyle == 'triple' ? '#FB6350' : '' }">
        <span>triple</span>
      </div>
  </div>

  <template v-if="currentMode == 'search'">
    
    <div>
      <form @submit.prevent="searchAnime">
        <input v-model="query" placeholder="Search for Anime..." ref="searchInput">
        <button type="submit">Search</button>
      </form>
      <span v-if="fetchedData?.length > 0"><br><br>Found: {{ fetchedData.length }}</span>
      
    </div>
  </template>

  <template v-if="currentMode == 'popular'">
    <div @click="fetchPopularAnime()" style="position: absolute; left: 0; bottom:-30px; transform: translateY(0%); display: block;" class="button">Get More </div>
  </template>

  <template v-if="currentMode == 'trending'">
    
    <div @click="fetchRandomAnime()" style="position: absolute; right: 0; transform: translateY(-330%);" class="button">Get Random </div>
    
    
  </template>

  <template v-if="currentMode == 'profile'">
    <div v-if="!isProgressMax" class="progress-bar" :style="{ width: progressBarWidth }"></div>
    <!-- <vue-progress-bar v-if="loading"></vue-progress-bar> -->
    <div>

      <div class="tab-wrapper">
        <div @click="myAnimeListTab = 'past'; fetchAnimeList() " class="tab"  :style="{ background: myAnimeListTab == 'past' ? '#FB6350' : '' }">
          <span>Past: {{ myAnimeList.filter(anime => anime.status === 'past').length }}</span>
        </div>
        <div @click="myAnimeListTab = 'current'; fetchAnimeList() " class="tab"  :style="{ background: myAnimeListTab == 'current' ? '#FB6350' : '' }">
          <span>Current: {{ myAnimeList.filter(anime => anime.status === 'current').length }}</span>
        </div>
        <div @click="myAnimeListTab = 'future'; fetchAnimeList() " class="tab"  :style="{ background: myAnimeListTab == 'future' ? '#FB6350' : '' }">
          <span>Future: {{ myAnimeList.filter(anime => anime.status === 'future').length }}</span>
        </div>
      </div>

      <h2>My anime list</h2>
    </div>
  </template>

  <div class="list-container">
    <template v-if="fetchedData.length && !(currentMode === 'profile' && myAnimeListTab === 'past')">

      <template v-if="listStyle == 'detail'">
        <ul class="detail-row-list-container">
            <template v-for="(anime, index)  in fetchedData" :key="anime.id" >
              <li 
                :id="`anime-${anime.id}`"
                
                ref="setAnimeRef"
                data-aos="fade-up"
                @touchstart="startPress(anime, $event)" 
                @touchend="cancelPress"
                @touchmove="moveImage($event)"
              >
                <div class="image-container">
                  <img :src="anime.coverImage.large" :alt="anime.title.romaji"/>
                  <div class="overlay">
                    <p class="anime-title">{{ anime.title.native }}</p>
                    <span class="studio">{{ anime.studios?.nodes[0]?.name }}</span>
                  </div>
                </div>

                <div class="anime-detail">

                  <strong>No.{{ index+1 }}</strong> <span style="float: right;" v-if="anime.startDate?.year">{{ anime.startDate?.year }} - {{ anime.endDate?.year }}</span>
                  <p v-html="anime.description" class="description"></p>

                  <hr>

                  <template v-for="(genre)  in anime.genres" :key="genre" >
                    <span class="genre">{{ genre }}, </span>
                  </template>

                  

                  <template v-if="anime.format == 'TV'">
                    <hr>

                    <template v-if="currentMode == 'profile' && myAnimeListTab == 'current'">
                      <div class="anime-progress-box">
                        <div class="anime-progress-top" v-if="anime.episodes">
                          <vue3-autocounter class="counter" ref='counter' :startAmount='0'  suffix='%' :endAmount="anime.progress" :duration='1.25'  separator=',' :autoinit='true' />
                        </div>
                        <div class="anime-progres-middle">
                          <div class="anime-progress-container">
                            <div class="anime-progress-bar" :style="{ width: anime.hasAppeared ? (checkProgress(anime)/anime.episodes * 100).toFixed(1) + '%' : '0%' }"></div>
                          </div>
                        </div>
                        <div class="anime-progres-bottom">
                          <div> 
                            <i class="fas fa-pencil-alt" @click="isUserTyping = true;animeProgressInputId = anime.id"></i>
                            <vue3-autocounter class="number-counter" ref='counter' :startAmount='0'  suffix=' EPs' :endAmount="anime.counterEndAmount" :duration='1.25'  separator=',' :autoinit='true' />
                            
                          </div>
                          <div>{{ anime.episodes }} Eps<i class="fa-solid fa-trash" style="color: #FB6350; margin: 0 0 0 7.5px;" @click="removeAnime(anime)" ></i></div>
                        </div>
                      </div>

                      
                      
                    </template>
                    

                    

                    <template v-else>
                      <div style="float:right; color: black;">{{ anime.episodes }} Eps<i class="fa-solid fa-trash" style="color: #FB6350; margin: 0 0 0 7.5px;" @click="removeAnime(anime)" ></i></div>
                      <!-- <span v-if="anime.episodes" style="float:right; color: black;">EPs: {{ anime.episodes }}</span> -->
                    </template>

                  </template> 

                </div>

                
                <!-- <span style="float: right;">{{ getAnimeFavourites(anime.favourites) }}</span> -->
              </li>
            </template>
        </ul>
      </template>

      <template v-if="listStyle == 'single'">
        <ul class="single-row-list-container">
            <template v-for="(anime, index)  in fetchedData" :key="anime.id" >
              <li 
                data-aos="fade-up"
                @touchstart="startPress(anime, $event)" 
                @touchend="cancelPress"
                @touchmove="moveImage($event)"
              >
                <div class="image-container">
                  <img :src="anime.coverImage.medium" :alt="anime.title.romaji"/>
                </div>
                No.{{ index+1 }}. {{ anime.title.native }}. <span style="float: right;">{{ getAnimeFavourites(anime.favourites) }}</span>
              </li>
            </template>
        </ul>
      </template>

      <template v-if="listStyle == 'double'">
        <ul class="binary-list-container">
            <template v-for="(anime, index)  in fetchedData" :key="anime.id" >
              <li data-aos="fade-up" :data-aos-delay="((index % 2) * 100) + 0"  data-aos-duration="1000"  
              >
              <div class="image-container">
                <img :src="anime.coverImage.large" :alt="anime.title.romaji" @touchstart="startPress(anime, $event)" 
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
                  <br>
                  <i class="fa-solid fa-trash" style="color: #FB6350; display: block;text-align: right;" @click="removeAnime(anime)" v-if="currentMode == 'profile'"></i>
                    
                  </div>
                  
                
              </li>
            </template>
          </ul>
      </template>

      <template v-if="listStyle == 'triple'">
        <ul class="triple-row-list">
          <template v-for="(anime, index)  in fetchedData" :key="anime.id" >
            <li 
              data-aos="fade-up" :data-aos-delay="((index % 3) * 100) + 0"  data-aos-duration="1000"  
              @touchstart="startPress(anime, $event)" 
              @touchend="cancelPress($event)"
              @touchmove="moveImage($event)"
            >
              <img :src="anime.coverImage.large" :alt="anime.title.romaji"/>
  
              <p @click="getDetail(anime)">{{index +1}}. {{ anime.title.native }}</p>
              <div class="star-rating">
                  <div class="stars-outer">
                      <div class="stars-inner" :style="{ width: `${anime.averageScore}%` }"></div>
                      
                      
                  </div>
                  <span>EP:{{ anime.episodes }}</span>
              </div>
              
            </li>
          </template>
        </ul>
      </template>

    </template>

    <template v-else>
      <template v-for="(element, index) in timelineData" :key="index">
        <!-- Display the year -->
        <span>{{ element.year }}</span><br>
        <ul class="triple-row-list" style="margin-top: 10px; margin-bottom: 15px">
          <!-- Loop through the IDs associated with the year -->
          <template v-for="(animeId, animeIndex) in element.ids" :key="animeId">
            <li 
              data-aos="fade-up" 
              :data-aos-delay="((animeIndex % 3) * 100) + 0" 
              data-aos-duration="1000"  
              @touchstart="startPress(foundAnime(animeId), $event)" 
              @touchend="cancelPress($event)"
              @touchmove="moveImage($event)"
            >
              <!-- Anime cover image -->
              <img 
                :src="foundAnime(animeId)?.coverImage.large" 
                :alt="foundAnime(animeId).title.romaji"
              />
              <!-- Anime title -->
              <p @click="getDetail(foundAnime(animeId))">
                {{ animeIndex + 1 }}. {{ foundAnime(animeId).title.native }}
              </p>
            </li>
          </template>
        </ul>
      </template>

    </template>
  </div>



  

  <div v-if="showModal" class="modal">
    <!-- <div id="center-area" class="area">Center</div> -->
    <div id="top-area" class="area">Currently</div>
    <div id="right-area" class="area">Future</div>
    <!-- <div id="bottom-area" class="area">Bottom</div> -->
    <div id="left-area" class="area">Past</div>
  </div>

  <div v-if="isUserTyping" style="z-index:99; background-color: grey; border: 2px solid black; border-radius: 10px; padding: 15px; position: fixed; top: 50%; left: 50%; transform: translate(-50%,-50%); height: 30%;">
    <label for="progressInput">Enter Anime Progress:</label>
    <input type="number" id="progressInput" v-model="animeProgressInput">
    <br><br>
    <button @click="updateProgress()">Update Progress</button>
    <br><br>
    <button @click="isUserTyping = false">Cancel</button>

    <br><br><br><br>
    <input type="number" v-model="tempTotalEpisodes">
    <br><br>
    <button @click="updateTempTotal()">Update <br>tempTotalEpisodes</button>

  </div>

  <div v-if="showCircle" 
         :style="{ background: `url(${selectedImage}) center/cover`, 
                   transform: `translate(${imgPosition.x}px, ${imgPosition.y}px)` }" 
         class="circle"
  ></div>
    

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
  // import CountTo from 'vue-count-to';
  import Vue3autocounter from 'vue3-autocounter';

  export default {
    name: "TrendingAnime",
    components: {
      // CountTo,
      'vue3-autocounter': Vue3autocounter,
    },

    data() {
      return {
        currentMode: 'trending',
        fetchedData: [],
        loading: false,
        testNum: 3,

        composing: false,

        ratingPercent: Number,

        isMenuOpen: false,

        rotations: {
          pie1: -90,
          pie2: -67.5,
          pie3: -45,
          pie4: -22.5,
        },

        showCircle: false,
        selectedImage: null,
        selectedAnime: null,
        pressTimer: null,
        imgPosition: { x: 0, y: 0 },
        lastPosition: { x: 0, y: 0 },

        showModal: false,
        scrollPosition: 0,

        query: '',
        searchList: null,
        totalResults: 0,
        animeCount: 0,
        mangaCount: 0,

        progressBarWidth: '0%',
        isProgressMax: false,

        popularPage: 1,

        myAnimeList: [],
        myProgressList: [],
        animeElements: [],
        oveserver: null,

        timelineData: [],


        listStyle: "triple",

        myAnimeListTab: 'current',

        testVal: 20,

        isUserTyping: false,
        animeProgressInput: null,
        animeProgressInputId: null,

        tempTotalEpisodes: null,




      };
    },
    methods: {
      async fetchTrendingAnime() {
        this.loading = true;
        this.startLoading(); // Start the custom progress bar
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
                large
                medium
                large
                color
              }

              format
              duration
              volumes

                
              }
            }
          }
        `;
        // studios
        // endDate

        const variables = {
          page: 1,
          perPage: 50
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
          console.log(this.fetchedData)
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
        // console.log(randomPage)
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

        this.showCircle = false;
        this.showModal = false

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
          
        }, 500); // Delay to ensure users see the progress bar hit 100%
        

        if(newMode == 'search') this.searchList = null
        if(newMode == 'trending') this.fetchTrendingAnime()
        if(newMode == 'popular') {
          this.popularPage =1
          this.fetchPopularAnime()
        }

        if(newMode == 'profile') {
          this.fetchAnimeList()
        }
        
      },


      startPress(anime, event) {
        if(this.currentMode == 'profile') return
        
        this.pressTimer = setTimeout(() => {
          

          this.showCircle = true;
          this.showModal = true;
          this.selectedImage = anime.coverImage.large;
          this.selectedAnimeId = anime.id
          this.imgPosition = {
            x: event.touches[0].clientX - 50, // Assuming circle radius 50px
            y: event.touches[0].clientY - 50,
          };
        }, 750); // .5 second
      },
      cancelPress(event) {
        // if(this.currentMode == 'trending') return
        clearTimeout(this.pressTimer);
        this.showCircle = false;

        if(!this.showModal) return


        this.showModal = false;

        


       // Function to check if the touch end is within an area
        function isTouchInArea(touchX, touchY, rect, status) {
          if(status == 'current'){
            return touchX >= rect.left && touchX <= rect.right && touchY >= 0 && touchY <= 150;
          }
            return touchX >= rect.left && touchX <= rect.right && touchY >= rect.top && touchY <= rect.bottom;
        }

        // Get the touch end position
        const touchEndX = event.changedTouches[0].clientX;
        const touchEndY = event.changedTouches[0].clientY;

        // alert(`${touchEndX},${touchEndY}`)

        // Get the areas and their positions
        const areas = {
            current: document.getElementById('top-area').getBoundingClientRect(),
            future: document.getElementById('right-area').getBoundingClientRect(),
            past: document.getElementById('left-area').getBoundingClientRect(),
        };


        // Check each area
        for (const [areaName, rect] of Object.entries(areas)) {
            if (isTouchInArea(touchEndX, touchEndY, rect,areaName)) {
                // alert(areaName)
                
                const status = areaName
                const animeExists = this.myAnimeList.some(anime => anime.id === this.selectedAnimeId && anime.status === status);
                alert(`adding to ${status}`)
                

                if (!animeExists) {
                  
                  this.myAnimeList.push({ id: this.selectedAnimeId, status, lastUpdated:  Date.now() });
                } else {
                  // Handle the rejection case, maybe with an alert or error message
                  console.log('Anime with the same ID and status already exists.');
                }
                // this.myAnimeList.push({ id: this.selectedAnimeId, status });
                break; // Assuming only one area can be touched at a time
            }
        }

        // console.table(this.groupedAnime)
        
      },
      moveImage(event) {
        if (!this.showCircle) return;
        

        const currentX = event.touches[0].clientX - 50;
        const currentY = event.touches[0].clientY - 50;
        // console.log(currentX,currentY)

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
        this.$refs.searchInput.blur();
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
          this.fetchedData = jsonResponse.data.Page.media
          console.log(this.searchList)
          // console.table(this.fetchedData)
        } catch (error) {
          console.error('Error fetching data:', error);
        } finally {
          this.loading = false;
        }
      },
      handleEnter() {
        if (!this.composing) {
          this.searchAnime();
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
              format
              duration
              volumes

                
              }
            }
          }
        `;
        // studios
        // endDate

        const variables = {
          page: this.popularPage,
          perPage: 30
        };

        // console.log(this.popularPage)
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

      async fetchAnimeList() {
        if(this.myAnimeList.length < 1) return
        this.loading = true;
        this.startLoading(); // Assuming you have a custom progress bar method

        const query = `
          query ($ids: [Int]) {
            Page {
              media(id_in: $ids, type: ANIME) {
                id
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
          ids: this.filteredMyAnimeList.map(anime => anime.id)
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

          this.sortMyList()

          if(this.myAnimeListTab == 'past') this.sortByRelease()
          this.checkVisibility()

          console.log(this.fetchedData)
          
        } catch (error) {
          console.error('Error fetching data:', error);
        } finally {
          this.loading = false;
          await new Promise(resolve => setTimeout(resolve, 500));
          window.scrollBy(0, 100);
          window.scrollBy(0, -100);
          
        }
      },

      removeAnime(animeToRemove) {
        const animeId = animeToRemove.id;
        // Find the index of the anime with the given id and status in myAnimeList
        const index = this.myAnimeList.findIndex(anime => anime.id === animeId && anime.status === this.myAnimeListTab);

        // If the anime with the given id and status is found, remove it
        if (index !== -1) {
          const confirmMessage = `Are you sure you want to remove anime with ID ${animeToRemove.title.native}`;
          if (confirm(confirmMessage)) {
            this.myAnimeList.splice(index, 1);
            console.log(`Removed anime with id ${animeId} and status ${this.myAnimeListTab}`);
            this.fetchAnimeList()
          }
        } else {
          console.log(`Anime with id ${animeId} and status ${this.myAnimeListTab} not found in myAnimeList`);
        }
      },

      resetLocaldata(){
        const confirmMessage = `Are you sure you want to delete local Data?`;
        if (confirm(confirmMessage)) {
          localStorage.removeItem('localMyAnimeList'); // Remove the item from localStorage
          location.reload(); // Reload the page
        }
      },

      disableScroll() {
        this.scrollPosition = window.pageYOffset; // Store the current scroll position
        document.body.style.overflow = 'hidden';
        document.body.style.position = 'fixed';
        document.body.style.top = `-${this.scrollPosition}px`; // Offset the body's position
        document.body.style.width = '100%';
      },

      enableScroll() {
        document.body.style.removeProperty('overflow');
        document.body.style.removeProperty('position');
        document.body.style.removeProperty('top');
        document.body.style.removeProperty('width');
        window.scrollTo(0, this.scrollPosition); // Restore the scroll position
      },

      
      checkVisibility() {
        if(this.fetchedData?.length < 1) return
        // console.log('hey');
        this.fetchedData.forEach(anime => {
          // const el = this.$refs[anime.id];
          const el = document.getElementById(`anime-${anime.id}`);
          
          
          if(!el) return

          if(!anime.hasAppeared){
            anime.counterEndAmount = 0
            anime.progress = 0
          }


          // if(el) console.log('exis');
          if (this.isElementInViewport(el)) {
            anime.hasAppeared = true;

            anime.counterEndAmount = this.checkProgress(anime)
            if(anime.episodes) {
              // anime.progress = (10/anime.episodes * 100).toFixed(1)
              anime.progress = (this.checkProgress(anime)/anime.episodes * 100)
            }
          }
        });
      },
      isElementInViewport(el) {
        const rect = el.getBoundingClientRect();
        // console.log('checkigns');
        return (
          rect.top >= 0 &&
          rect.left >= 0 &&
          rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) &&
          rect.right <= (window.innerWidth || document.documentElement.clientWidth)
        );
      },

      checkProgress(anime) {
        // Assuming myAnimeList is an array of anime objects with an 'id' property
        const foundAnime = this.myAnimeList.find(item => item.id === anime.id);
        
        if (foundAnime) {
          if (foundAnime.progress) return foundAnime.progress; // Return the progress if the anime is found
          return 1;
        } else {
          return 1; // Return 1 if the anime is not found
        }
      },

      updateProgress() {
        const foundIndex = this.myAnimeList.findIndex(item => item.id === this.animeProgressInputId);

        if (foundIndex !== -1) {
          this.myAnimeList[foundIndex].progress = this.animeProgressInput;
          this.myAnimeList[foundIndex].lastUpdated =  Date.now();
          // console.log( Date.now())
          
          this.checkVisibility()
          // console.log(`Updated Anime Progress: ${this.animeProgressInput}`);
        } else {
          console.log("Anime not found in myAnimeList.");
        }

        this.animeProgressInput = null;
        this.animeProgressInputId = null;
        this.isUserTyping = false;
        this.sortMyList()
      },

      updateTempTotal() {
        const foundIndex = this.myAnimeList.findIndex(item => item.id === this.animeProgressInputId);

        if (foundIndex !== -1) {
          this.myAnimeList[foundIndex].tempTotalEpisodes = this.tempTotalEpisodes;
          // this.myAnimeList[foundIndex].lastUpdated =  Date.now();
          // console.log( Date.now())
          
          this.checkVisibility()
          // console.log(`Updated Anime Progress: ${this.animeProgressInput}`);
        } else {
          console.log("Anime not found in myAnimeList.");
        }

        // this.animeProgressInput = null;
        this.tempTotalEpisodes = null;
        this.isUserTyping = false;
        this.sortMyList()
      },

      sortMyList(){
        this.fetchedData.forEach(fetchedAnime => {
          const foundAnime = this.filteredMyAnimeList.find(filteredAnime => filteredAnime.id === fetchedAnime.id);
          if (foundAnime) {
            fetchedAnime.lastUpdated = foundAnime.lastUpdated;
            if(!fetchedAnime.episodes && foundAnime.tempTotalEpisodes){
              fetchedAnime.episodes = foundAnime.tempTotalEpisodes
            }
          }
        });
        this.fetchedData = this.fetchedData.sort((a, b) => {
          const lastUpdatedA = a.lastUpdated || 0; // Use 0 if 'lastUpdated' is not present
          const lastUpdatedB = b.lastUpdated || 0; // Use 0 if 'lastUpdated' is not present

          return lastUpdatedB - lastUpdatedA; // Sort in descending order
        });
      },
      sortByRelease(){
        this.fetchedData =this.fetchedData.sort((a, b) => {
          const yearDiff = a.startDate.year - b.startDate.year;
          if (yearDiff !== 0) {
            return yearDiff; // Sort by year first
          } else {
            return a.startDate.month - b.startDate.month; // If years are the same, sort by month
          }
        });
        this.createTimelineData();
      },
      
      createTimelineData() {
        this.timelineData = [];
        let timelineMap = new Map();
      
        this.fetchedData.forEach(item => {
          const yearKey = item.startDate.year; // Group by year only
          if (!timelineMap.has(yearKey)) {
            timelineMap.set(yearKey, {
              year: yearKey,
              ids: []
            });
          }
          timelineMap.get(yearKey).ids.push(item.id);
        });
      
        this.timelineData = Array.from(timelineMap.values());
        console.table(this.timelineData);
      },

      foundAnime(id) {
        return this.fetchedData.find(anime => anime.id === id);
      },



    },

    watch: {
      
      showModal(newValue) {
        if (newValue) {
          this.disableScroll();
        } else {
          this.enableScroll();
        }
      },
      myAnimeList: {
        deep: true,
        handler(newValue) {
          localStorage.setItem('localMyAnimeList', JSON.stringify(newValue));
        }
      },
      
    },

    computed: {
      filteredMyAnimeList() {
        return this.myAnimeList
          .filter(anime => anime.status === this.myAnimeListTab);
      },
      
    },

    async mounted() {
      console.clear()
      AOS.init();

      const localMyAnimeList = localStorage.getItem('localMyAnimeList');
      if (localMyAnimeList) {
        // If it exists, set it to the component's data
        this.myAnimeList = JSON.parse(localMyAnimeList);
        console.log('Local data "animeList" exists:', this.myAnimeList);
      } else {
        console.log('Local data "animeList" does not exist.');
      }

      this.myProgressList = [
        {id: 11061, watched: 50}
      ]
      
      // this.fetchTrendingAnime();
      // this.currentMode = 'profile'
      // this.currentMode = 'trending'
      this.currentMode = 'popular'

      // this.listStyle = 'triple'
      this.listStyle = 'triple'
      this.fetchPopularAnime()
      
      // this.myAnimeList =[16498,113415,11061]
      this.myAnimeListTab = 'past'
      // await this.fetchAnimeList()

      // this.sortByRelease()

      // this.isMenuOpen = true

      this.checkVisibility(); // Initial check
      window.addEventListener('scroll', this.checkVisibility);
    },

    beforeUnmount() {
      window.removeEventListener('scroll', this.checkVisibility);
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

    max-width: 750px;
    margin:auto;
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

  .triple-row-list{
    padding-inline-start: unset;

    display: grid;
    grid-template-columns: 31% 31% 31%;
    justify-content: space-between;

    width: 95%;
    margin:auto;

    margin-bottom: 100px;

    /* background: red; */
  }

  .triple-row-list li{
    list-style: none;
  }

  .triple-row-list li img{
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

  .triple-row-list li p{
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

  .radial-menu{
    
  }

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

    /* pointer-events: none; */
    user-select: none;
    -webkit-user-select: none;

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
    opacity: .8;
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

  .detail-row-list-container{
    margin: 0;
    margin-bottom: 120px;
    padding-left: 0;
    list-style: none;
  }

  .detail-row-list-container li {
    display: grid;
    grid-template-columns: 45% 52.5%;
    justify-content: space-between;
    /* justify-content: space-around; */
    align-items: center;
    border: 1.5px solid black;
    margin: 10px auto;

    border-radius: 10px;
    overflow: hidden;

  }

  .detail-row-list-container .image-container{
    display: flex;
    justify-content: center;
    align-items: center;
    overflow: hidden; 
    position: relative;
    
    /* border: 2px solid red; */

    aspect-ratio: 2/3;
    width: 100%;
    overflow: hidden; 
    position: relative;
    /* height: 200px !important; */
  }

  .detail-row-list-container li img{
    height: 100%;
    /* width: 100%; */
    /* height: auto; */
    /* height: 100px;
    margin: 0;
    margin-right: 10px;

    display: block; 
    vertical-align: bottom; */

    position: absolute;
    top: 50%;
    left:50%;
    transform: translate(-50%,-50%);
  }

  .detail-row-list-container .image-container .overlay{
    position: absolute;
    bottom: 0;
    display: block;
    color: black;
    /* padding: 5px; */
    width:100%;
    background: rgba(251, 99, 80, 0.8);
    /* opacity: .8; */
    z-index: 100;
  }

  .detail-row-list-container .image-container .overlay .anime-title{
    font-weight: bold;
    /* color: white; */
    display: block;
    width: 90%;
    margin: 10px auto;
    font-size: 16px;
    color: white;
  }

  .detail-row-list-container .image-container .overlay .studio{
    display: block;
    width: 90%;
    margin: 10px auto;
    font-weight: bold;
    font-size: 15px;
    /* color: white; */
  }

  .detail-row-list-container .anime-detail{
    padding: 5px 12.5px;
  }

  .detail-row-list-container .anime-detail .genre{
    font-size: .8em;
    line-height: 1.2;
  }

  .detail-row-list-container .description{
    transition: all 2s ease;
    margin: 10px 0;
    line-height: 1.2em;
    /* max-height: 15em; */
    overflow: hidden;
    display: -webkit-box;
    -webkit-line-clamp: 4;
    -webkit-box-orient: vertical;
    text-overflow: ellipsis;
    white-space: normal;

    color: grey;
    font-size: .85em;
  }

  .anime-progress-box{
    /* display: grid;
    grid-template-columns: 10% 75% 15%;
    justify-content: space-between;
    align-items: center; */
  }

  .anime-progress-top{
    text-align: right;
    display: block;
  }

  .anime-progres-bottom{
    display: flex;
    justify-content: space-between;
    font-size:.9em;
  }

  .number-counter{
    /* display: inline-block;
    min-width: 55px; */
  }

  .anime-progres-bottom i{
    margin-right: 7.5px;
    
  }

  .item-progress-container {
    margin-bottom: 10px;
  }



  .anime-progress-container {
    width: 100%;
    height: 5px;
    margin: 5px auto 15px;
    background-color: lightgray;
    position: relative;
  }

  .anime-progress-bar {
    position: absolute;
    left: 0;
    display: inline-block;
    height: 100%;
    background-color: #FB6350;
    width: 0%;
    max-width: 100%;
    transition: all 1.25s ease-in-out;
  }


  .single-row-list-container{
    margin: 0;
    margin-bottom: 120px;
    padding-left: 0;
    list-style: none;
  }

  .single-row-list-container li {
    display: grid;
    grid-template-columns: 20% 60% 10%;
    justify-content: space-between;
    align-items: center;
    border: 2px solid black;
    margin: 10px auto;

  }

  .single-row-list-container .image-container{
    display: flex;
    justify-content: center;
    align-items: center;
    /* width: 100%; */
    aspect-ratio: 1/1;
    overflow: hidden; 
    position: relative;
    height: 80px !important;

  }

  .single-row-list-container li img{
    /* height: 100px;
    margin: 0;
    margin-right: 10px;

    display: block; 
    vertical-align: bottom; */

    position: absolute;
    top: 50%;
    left:50%;
    transform: translate(-50%,-50%);
  }



  /* ------------------- */
  .modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5); /* Semi-transparent background */
    display: flex;
    justify-content: center;
    align-items: center;
    animation: fadeIn 0.5s; /* Smooth fade-in effect */

    /* pointer-events: none;
    user-select: none;
    -webkit-user-select: none; */
  }

  .area {
    position: absolute;
    display: block;
    width: 100px;
    aspect-ratio: 2/3;
    background: white;
    /* Additional styling for each area */
  }

  .no-scroll {
    overflow: hidden;
    height: 100%;
    width: 100%;
    position: fixed; /* This can help in some cases */
    /* background: green; */
  }


  #center-area { /* Center area styling */ }
  #top-area { top: 0; /* Top area styling */ }
  #right-area { right: 0; /* Right area styling */ }
  #bottom-area { bottom: 0; /* Bottom area styling */ }
  #left-area { left: 0; /* Left area styling */ }

  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }



  /* ----------- */
  .tab-wrapper{
    width: 90%; 
    margin: auto; 
    display: grid; 
    grid-template-columns: 30% 30% 30%; 
    text-align: center; 
    justify-content: space-between;
    margin-bottom: 20px;
  }

  .tab-wrapper .tab{
    border: 2px solid black;
    transition: all .75s ease-in-out;
  }


</style>
