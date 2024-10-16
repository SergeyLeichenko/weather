<template>
  <div class="tabs">
    <div class="tabs__nav">
      <div class="tabs__nav_tab" v-for="(item, index) in items" :key="item + index"
        :class="[index === active ? 'tabs__nav_tab--active' : '']" @click="activate(index)">{{ item }}</div>
    </div>
    <div class="tabs__content">

      <div v-if="active === 0" class="btn-add">
        <button class="btn-add-new" @click="openField" v-show="hideBtn">Додати нове місто</button>

        <form class="input-city" v-show="hideInputNewCity">
          <input type="text" name="city" placeholder="Введіть місто" autocomplete="on" v-model="newCity" />
          <div class="add-close">
            <button class="btn-new-city add" @click.prevent="addNewCity">Додати</button>
            <button class="btn-new-city" @click.prevent="hideBtn = true && !hideInputNewCity">Закрити</button>
          </div>
        </form>
      </div>

      <div v-if="active === 0" class="container-weathers">
        <div v-if="loading">
            <LoadingComponent />
        </div>
          
          <CardWeather 
            v-for="(item, i) in weathers" 
            :key="item + i"
            :weather="item"
            :showFavorite="item.active"
            :daysOfWeek="item.daysOfWeek"
            :temp="item.tempOfWeek"
            @addFavorite="addFavorite(item)"
            @removeCard="removeCard(i)"
            @getForFiveDays="getForFiveDays(item)"
          />
      </div>

      <div v-if="active === 1" class="container-weathers">
        <CardWeather 
          v-for="(item, i) in favorites" 
          :key="item + i"
          :weather="item"
          :showFavorite="item.active"
          :daysOfWeek="item.daysOfWeek"
          :temp="item.tempOfWeek"
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
      loading: false,
      newCity: '',
      hideInputNewCity: true,
      hideBtn: false
    }
  },
  created() {
    this.getUserLocation()
  },
  methods: {
    openField() {
      this.hideInputNewCity = true
      this.hideBtn = false
      this.newCity = ''
    },
    activate(index) {
      this.active = index
    },
    addNewCity() {
      if (this.newCity === '') {
        return
      } else {
        this.hideInputNewCity = false
        this.hideBtn = true
      }
      this.getWeather()
    },
    // get weather with your city
    async getWeather() {
      this.loading = true

      let data = await axios
      .get(`${this.baseUrl}/data/2.5/weather`, {
        params: {
          q: this.newCity,
          appid: this.apiKey,
          lang: 'ru',
          units: 'metric'
        }
      })

      if (data) {
        this.loading = false
      }

      let sunset = data.data.sys.sunset
      let date = new Date()
      date.setTime(sunset)
      let sunset_date = date.getHours() + ":" + date.getMinutes() + ":" + date.getSeconds()

      let weather = {
        active: false,
        id: data.data.id,
        temp: data.data.main.temp,
        city: data.data.name,
        country: data.data.sys.country,
        pressure: data.data.main.pressure,
        sunset: sunset_date,
        lat: data.data.coord.lat,
        lon: data.data.coord.lon,
        icon: data.data.weather[0].icon + '.png',
        description: data.data.weather[0].description,
        daysOfWeek: [],
        tempOfWeek: []
      }

      let isCardAdd = false
        if (this.weathers.length) {
          this.weathers.map(function (elem) {
            if (data.data.id === elem.id) {
              alert('Місто вже додано')
              isCardAdd = true
            }
          })
          if (!isCardAdd) {
            this.weathers.push(weather)
          }
        } else {
          this.weathers.push(weather)
        }
      
      this.saveWeatherInLocalStore()
    },
    saveWeatherInLocalStore() {
      localStorage.setItem('weathers', JSON.stringify(this.weathers))
    },
    // add favorite card
    addFavorite(item) {
      if (this.favorites.length > 4) {
        alert('для додавання видаліть місто … тому що максимум 5')
      } else {
        this.weathers.forEach((el) => {
          if (el.city === item.city) {
            el.active = true
          }
        })

        let isCardExists = false
        if (this.favorites.length) {
          this.favorites.map(function (elem) {
            if (elem.city === item.city) {
              alert('Картку вже додано до обраного')
              isCardExists = true
            }
          });
          if (!isCardExists) {
            this.favorites.push({...item, active: true})
          }
        } else {
          this.favorites.push({...item, active: true})
        }

        localStorage.setItem("favorites", JSON.stringify(this.favorites))
        this.saveWeatherInLocalStore()
      }
    },
    // remowe card with weather
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
    // get weather for five days
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

        this.weathers.forEach((el) => {
          if (el.city === data.data.city.name) {
            el.daysOfWeek.push(weekdayName)
            el.tempOfWeek.push(Math.floor(item.main.temp))
          }
        })
      })
      this.saveWeatherInLocalStore()
    },
    // get user location
    getUserLocation() {
      const options = {
        enableHighAccuracy: true,
        timeout: 5000,
        maximumAge: 0
      }

      const success = async (pos) => {
        const crd = pos.coords

        const response = await axios.get(`https://api.geoapify.com/v1/geocode/reverse`, {
          params: {
            lat: crd.latitude,
            lon: crd.longitude,
            apiKey: "06a6d03278a14cb78c916513eb497077"
          }
        })

        let getYourCity = response.data.features[0].properties.city
        this.newCity = getYourCity
      }

      const error = (err) => {
        console.log(err.code + ' ' + err.message)
      }

      navigator.geolocation.getCurrentPosition(success, error, options)
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

.input-city {
  width: 320px;
  display: block;
  margin: auto;
}
.input-city input[type="text"] {
  width: 93%;
  padding: 10px;
  border-radius: 5px;
  margin-bottom: 10px;
  background-color: #c0c0c045;
}

.add-close {
  display: flex;
  justify-content: center;
}

.btn-new-city {
  background: #ddd;
  padding: 10px;
  width: 100px;
  border: 1px solid;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
}
.btn-new-city:hover {
  background: #235d9c;
  color: #fff;
}
.btn-new-city.add {
  margin-right: 20px;
}

@media screen and (max-width: 842px) {
  .tabs {
    padding: 2px;
  }
}

</style>