<template>
   <q-page class="flex column" :class="bgClass">
      <q-inner-loading :showing="searching" color="white" />
      <section class="col q-pt-lg q-px-md">
         <q-input
            v-model="searchText"
            borderless
            dark
            placeholder="Search"
            @keyup.enter="getWeatherBySearch"
         >
            <template v-slot:before>
               <q-icon name="my_location" @click="getLocation" />
            </template>

            <template v-slot:append>
               <q-btn
                  round
                  dense
                  flat
                  icon="search"
                  @click="getWeatherBySearch"
               />
            </template>
         </q-input>
      </section>

      <template v-if="weatherData">
         <section class="col text-white text-center">
            <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>
            <div class="text-h6 text-weight-light">
               {{ weatherData.weather[0].main }}
            </div>
            <div class="text-h1 text-weight-thin q-my-lg relative-position">
               <span>{{ temp }}</span>
               <span class="text-h4 relative-position degree">&deg;F</span>
            </div>
         </section>

         <section class="col text-center">
            <q-img :src="weatherImage" alt="Weather Image" width="100px" />
         </section>
      </template>

      <template v-else>
         <section class="col column text-center text-white">
            <div class="col text-h2 text-weight-thin">Quasar<br />Weather</div>
            <q-btn class="col" flat @click="getLocation">
               <q-icon left size="3em" name="my_location" />
               <div>Find My Location</div>
            </q-btn>
         </section>
      </template>

      <section class="col skyline"></section>
   </q-page>
</template>

<script>
import { computed, defineComponent, inject, ref } from 'vue'
import { useQuasar } from 'quasar'

export default defineComponent({
   name: 'Index',

   setup() {
      const $q = useQuasar()

      const axios = inject('axios')

      const searching = ref(false)
      const searchText = ref('')
      const weatherData = ref(null)
      const lat = ref(null)
      const lon = ref(null)

      const weatherApiKey = 'b6cd22634f8f1c9a74704828f7f0523e'
      const weatherApiBaseUrl = 'https://api.openweathermap.org/data/2.5'
      const geoApiBaseUrl = 'https://freegeoip.app/json/'

      const temp = computed(() => {
         return Math.round(weatherData.value.main.temp)
      })

      const weatherImage = computed(() => {
         const weatherApiBaseUrl = 'http://openweathermap.org/img/wn'
         return `${weatherApiBaseUrl}/${weatherData.value.weather[0].icon}@2x.png`
      })

      const bgClass = computed(() => {
         if (!weatherData.value) return null

         if (weatherData.value.weather[0].icon.endsWith('n')) {
            return 'bg-night'
         } else {
            return 'bg-day'
         }
      })

      const getLocation = async () => {
         searching.value = true
         try {
            if ($q.platform.is.electron) {
               const result = await axios.get(geoApiBaseUrl)
               lat.value = result.data.latitude
               lon.value = result.data.longitude
               getWeatherByCoords()
               searching.value = false
            } else {
               navigator.geolocation.getCurrentPosition((position) => {
                  lat.value = position.coords.latitude
                  lon.value = position.coords.longitude
                  getWeatherByCoords()
                  searching.value = false
               })
            }
         } catch (error) {}
      }

      const getWeatherBySearch = async () => {
         try {
            const endpoint = `${weatherApiBaseUrl}/weather?q=${searchText.value}&appid=${weatherApiKey}&units=imperial`
            const response = await axios.get(endpoint)
            weatherData.value = response.data
            console.log('weatherData', weatherData.value)
         } catch (error) {
            console.error('getWeatherBySearch', error.message)
            weatherData.value = null
         }
      }

      const getWeatherByCoords = async () => {
         try {
            const endpoint = `${weatherApiBaseUrl}/weather?lat=${lat.value}&lon=${lon.value}&appid=${weatherApiKey}&units=imperial`
            const response = await axios.get(endpoint)
            weatherData.value = response.data
            console.log('weatherData', weatherData.value)
         } catch (error) {
            console.error('getWeatherByCoords', error.message)
            weatherData.value = null
         }
      }

      return {
         searching,
         searchText,
         weatherData,
         temp,
         weatherImage,
         bgClass,
         getLocation,
         getWeatherBySearch,
      }
   },
})
</script>

<style lang="sass">
.q-page
   background: #00B4DB
   background: -webkit-linear-gradient(to bottom, #0083B0, #00B4DB)
   background: linear-gradient(to bottom, #0083B0, #00B4DB)

   &.bg-night
      background: #0f2027
      background: -webkit-linear-gradient(to bottom, #0f2027, #203a43, #2c5364)
      background: linear-gradient(to bottom, #0f2027, #203a43, #2c5364)

.degree
   top: -44px

.skyline
   flex: 0 0 100px
   background: url(/skyline.png)
   background-size: contain
   background-position: bottom
</style>
