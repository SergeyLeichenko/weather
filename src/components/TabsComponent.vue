<template>
  <div class="tabs">
    <div class="tabs__nav">
      <div class="tabs__nav_tab" v-for="(item, index) in items" :key="item + index"
        :class="[index === active ? 'tabs__nav_tab--active' : '']" @click="activate(index)">{{ item }}</div>
    </div>
    <div class="tabs__content">

      <div v-if="active === 0" class="btn-add">
        <button class="btn-add-new" @click="addNewCity">Додати нове місто</button>
      </div>

      <div v-if="active === 0" class="container-weathers">
        <div v-if="loading">
            <LoadingComponent />
        </div>
          <CardWeather 
            @getWeather="getWeather"
            :weather="this.weather"
            :showFavorite="false"
            :class="{opacityCard: loading }"
            :daysOfWeek="daysOfWeek"
            :temp="temp"
            @addFavorite="addFavorite(weather)"
            @getForFiveDays="getForFiveDays(weather)"
          />

          <CardWeather 
            v-for="(item, i) in this.weathers" 
            :key="item + i"
            :weather="item.weather"
            :hideForm="hideForm"
            :showFavorite="showFavorite"
            :daysOfWeek="item.daysOfWeek"
            :temp="item.temp"
            @addFavorite="addFavorite(item)"
            @removeCard="removeCard(i)"
            @getForFiveDays="getForFiveDays(item)"
          />
      </div>

      <div v-if="active === 1" class="container-weathers">
        <CardWeather 
          v-for="(item, i) in this.favorites" 
          :key="item + i"
          :weather="item.weather"
          :hideForm="hideForm"
          :showFavorite="showFavorite"
          :daysOfWeek="item.daysOfWeek"
          :temp="item.temp"
          @removeCard="removeCard(i)"
          @getForFiveDays="getForFiveDays(item)"
        />
      </div>

    </div>
  </div>
</template>

<script>
import CardWeather from "./CardWeather.vue"
import LoadingComponent from "./LoadingComponent.vue"
import axios from 'axios'
import api from '../api/api.json'

export default {
  components: {
    CardWeather,
    LoadingComponent
  },
  data() {
    return {
      baseUrl: api.url,
      apiKey: api.key,
      active: 0,
      items: [
        'Головна',
        'Обране'
      ],
      weathers: [],
      favorites: [],
      weather: null,
      hideForm: false,
      showFavorite: false,
      loading: false,
      daysOfWeek: [],
      temp: []
    }
  },
  methods: {
    activate(index) {
      this.active = index
    },
    async getWeather(city) {
      if (!city) return

      this.loading = true

      let data = await axios
      .get(`${this.baseUrl}/data/2.5/weather`, {
        params: {
          q: city,
          appid: this.apiKey,
          lang: 'ru',
          units: 'metric'
        }
      })

      if (data) this.loading = false

      let sunset = data.data.sys.sunset
      let date = new Date()
      date.setTime(sunset)
      let sunset_date = date.getHours() + ":" + date.getMinutes() + ":" + date.getSeconds()

      this.weather = {
        temp: data.data.main.temp,
        city: data.data.name,
        country: data.data.sys.country,
        pressure: data.data.main.pressure,
        sunset: sunset_date,
        lat: data.data.coord.lat,
        lon: data.data.coord.lon,
        icon: data.data.weather[0].icon + '.png',
        description: data.data.weather[0].description
      }

      if (this.weather) {
        this.isDisabled = false
      }
    },
    addNewCity() {
      this.isDisabled = true

      this.weather = {
        weather: this.weather,
        daysOfWeek: this.daysOfWeek,
        temp: this.temp
      }

      this.weathers.push(this.weather)

      this.weather = {}
      this.hideForm = false
      this.showFavorite = true
      localStorage.setItem('showFavorite', "true")

      this.saveWeatherInLocalStore()
      location.reload()
    },
    saveWeatherInLocalStore() {
      localStorage.setItem('weathers', JSON.stringify(this.weathers))
    },
    addFavorite(item) {
      if (this.favorites.length > 4) {
        alert('для додавання видаліть місто … тому що максимум 5')
      } else {
        this.showFavorite = true
        this.hideForm = false
        this.favorites.push(item)
        localStorage.setItem("favorites", JSON.stringify(this.favorites))
      }
    },
    removeCard(i) {
      if (confirm('Ви дійсно хочете видалити?')) {
        if (this.active === 0) {
          this.weathers.splice(i, 1)

          let arr = JSON.parse(localStorage.getItem('weathers'))
          arr.splice(i, 1)
          localStorage.setItem('weathers', JSON.stringify(this.weathers))
        } else {
          this.favorites.splice(i, 1)

          let arrFavorites = JSON.parse(localStorage.getItem('favorites'))
          arrFavorites.splice(i, 1)
          localStorage.setItem('favorites', JSON.stringify(this.favorites))
        }
      }
    },
    async getForFiveDays(item) {

      let data = await axios
        .get(`${this.baseUrl}/data/2.5/forecast`, {
          params: {
            lat: item.lat,
            lon: item.lon,
            appid: this.apiKey,
            lang: 'ru',
            units: 'metric',
          }
        })

      const dailyData = data.data.list.filter(reading => reading.dt_txt.includes("18:00:00"))

      dailyData.forEach((item) => {
        let ms = item.dt * 1000
        let weekdayName = new Date(ms).toLocaleString('ua', {weekday: 'long'})

        this.daysOfWeek.push(weekdayName)
        this.temp.push(Math.floor(item.main.temp))
      })
    }
  },
  mounted() {
    try {
      if (localStorage.getItem('weathers')) {
        this.weathers = JSON.parse(localStorage.getItem('weathers'))
      }
    } catch (err) {
      localStorage.removeItem('weathers')
    }

    try {
      if (localStorage.getItem('favorites')) {
        this.favorites = JSON.parse(localStorage.getItem('favorites'))
      }
    } catch (err) {
      localStorage.removeItem('favorites')
    }

    try {
      if (localStorage.getItem('showFavorite')) {
        this.showFavorite = localStorage.getItem('showFavorite')
      } 
    } catch (err) {
      localStorage.removeItem('showFavorite')
    }

    
  }
}
</script>

<style scoped>

.opacityCard {
  opacity: 0.3;
}
.tabs {
  height: 100%;
  max-width: 1200px;
  margin: auto;
  box-sizing: border-box;
}

.tabs__nav {
  display: flex;
}

.tabs__nav_tab {
  padding: 1rem;
  margin: 0 0.5rem 0 0;
  background: #fff;
  border-radius: 0.5rem 0.5rem 0 0;
  box-shadow: inset 0 -1rem 0.75rem -1rem rgba(0, 0, 0, 0.25);
  cursor: pointer;
  opacity: 0.75;
  transition: 100ms linear all;
}

.tabs__nav_tab--active {
  opacity: 1;
  box-shadow: none;
}

.tabs__content {
  background: #fff;
  padding: 1rem;
  border-radius: 0 0.5rem 0.5rem 0.5rem;
}

.container-weathers {
  position: relative;
  display: flex;
  flex-wrap: wrap;
}

.btn-add {
  margin-bottom: 20px;
}

.btn-add-new {
  background: transparent;
  border: 1px solid black;
  padding: 10px;
  font-weight: 600;
  cursor: pointer;
}

@media screen and (max-width: 842px) {
  .tabs {
    padding: 2px;
  }
}

</style>