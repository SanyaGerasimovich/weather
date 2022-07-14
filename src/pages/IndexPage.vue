<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        label="Search"
        label-color="grey-2"
        dark
        borderless
      >

        <template v-slot:before>
          <q-icon
            @click="getLocation"
            name="place"
          />
        </template>

        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="getWeatherBySearch"/>
        </template>

      </q-input>
    </div>

    <template v-if="weatherData">

      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-md relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img :src="`https://openweathermap.org/img/wn/${ weatherData.weather[0].icon }@2x.png`" alt="Bill">
      </div>

    </template>
    <template v-else>
      <div class="col column text-white text-center">
        <div class="col text-h2 text-weight-thin">
          Quasar<br>Weather
        </div>
        <q-btn
          @click="getLocation"
          class="col"
          flat
        >
          <q-icon left size="2.5em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>
    <div class="col town">

    </div>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'IndexPage',
  data () {
    return {
      search: '',
      weatherData: null,
      lat: null,
      lon: null,
      apiUrl: 'https://api.openweathermap.org/data/2.5/weather',
      apiKey: '96b48801d3d6824bf25daf08e3420d0f',
    }
  },
  computed: {
    bgClass() {
      if( this.weatherData ) {
        if ( this.weatherData.weather[0].icon.endsWith('n') ) {
          return 'bg-night'
        } else {
          return 'bg-day'
        }
      }
    }
  },
  methods: {
    getLocation () {
      this.$q.loading.show()
      if(this.$q.platform.is.electron) {
        this.$axios.get('https://ipapi.co/json/')
          .then(response => {
            this.lat = response.data.latitude
            this.lon = response.data.longitude
            this.getWeatherByCoords()
          })
      } else {
        navigator.geolocation.getCurrentPosition((position) => {
          console.log('position :', position)
          this.lat = position.coords.latitude
          this.lon = position.coords.longitude
          this.getWeatherByCoords()
        })
      }
    },
    getWeatherByCoords () {
      this.$axios.get(
        `${ this.apiUrl }?lat=${ this.lat }&lon=${ this.lon }&exclude=hourly,daily&appid=${this.apiKey}&units=metric`
      ).then(response => {
        console.log('response :', response)
        this.weatherData = response.data
        console.log(response.data)
      }).catch(e => console.log(e))
        .finally(() => this.$q.loading.hide() )
    },
    getWeatherBySearch () {
      this.$q.loading.show()
      this.$axios.get(
        `${ this.apiUrl }?q=${ this.search }&appid=${ this.apiKey }&units=metric`
      ).then(response => {
        console.log('response :', response)
        this.weatherData = response.data
        console.log(response.data)
      }).catch(e => console.log(e))
        .finally(() => this.$q.loading.hide() )
    }
  }
})
</script>

<style>
.q-page{
  background: linear-gradient(to bottom, #136a8a, #267871);
}
.bg-day{
  background: linear-gradient(to bottom, #00b4db, #0083b0);
}
.bg-night{
  background: linear-gradient(to bottom, #232526, #414345);
}
.degree{
  top: -44px;
}
.town{
  flex: 0 0 100px;
  background: url("../assets/town.png") center bottom;
  background-size: contain;
}
</style>
