<template>
  <div class="infoWeath" @click="$emit('getIdCard')">
    <div class="favorite" v-if="showFavorite">
      <button class="btn" @click="removeCard">Видалити</button>
      <button class="btn" :class="{ 'active-favorite': isActiveFavirite }" @click="addFavorite">В Обране</button>
    </div>
    <div class="getWeather">
      <form class="form" @submit.prevent="getWeather" v-if="hideForm">
        <input type="text" name="city" placeholder="Введіть місто" autocomplete="on" v-model="city" />
        <button class="btn">Отримати погоду</button>
      </form>

      <div class="info-list" v-if="weather">
        <div class="temp" v-if="weather.city">

          <div class="icon">
            <img :src="`${baseUrl}/img/w/${weather.icon}`" />
            <p>{{ weather.description }}</p>
          </div>
          <p>{{ Math.floor(weather.temp) }} <sup>o</sup>C</p>
        </div>

        <p>Місце знаходження: {{ weather.city }} {{ weather.country }}</p>
        <p>Тиск: {{ weather.pressure }} мм</p>
        <p>Захід сонця: {{ weather.sunset }}</p>
      </div>
      <div class="buttons">
        <button class="btn" @click="getForFiveDays">на 5 днів</button>
      </div>

      <div class="line-chart">
        <Bar :chart-data="chartData"/>
      </div>

    </div>
  </div>
</template>

<script>
import axios from 'axios'
import { Bar } from 'vue-chartjs'
import { Chart, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'
import api from '../api/api.json'

Chart.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)

export default {
  components: { Bar },
  props: {
    weather: {},
    hideForm: {
      type: Boolean,
      default: true
    },
    showFavorite: {
      type: Boolean,
      default: false
    },
    daysOfWeek: {
      type: Array,
      default: () => []
    },
    temp: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      baseUrl: api.url,
      city: null,
      favorites: [],
      isActiveFavirite: false,
      chartData: {
        labels: this.daysOfWeek,
        datasets: [
          {
            label: 'Погода',
            data: this.temp,
            borderColor: '#235d9c',
            pointBackgroundColor: 'black',
            borderWidth: 1,
            pointBorderColor: 'black',
          }
        ]
      }
    }
  },
  created() {
    this.getLocation()
  },
  methods: {
    getWeather() {
      this.$emit('getWeather', this.city)
      this.city = ''
    },
    addFavorite() {
      this.$emit('addFavorite')
      this.isActiveFavirite = true
    },
    removeCard() {
      this.$emit('removeCard')
    },
    getForFiveDays() {
      this.$emit('getForFiveDays')
    },
    getLocation() {
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
        this.city = getYourCity
      }

      const error = (err) => {
        console.log(err.code + ' ' + err.message)
      }

      navigator.geolocation.getCurrentPosition(success, error, options)
    }
  }
}
</script>

<style>
.btn {
  border: 2px solid #103a65;
  border-bottom: 4px solid #103a65;
  padding: 8px 20px;
  margin: 0 2px;
  border-radius: 2px;
  cursor: pointer;
  background-color: #235d9c;
  color: #fff;
  outline: none !important;
}

.infoWeath {
  min-width: 40%;
  /* width: 40%; */
  height: auto;
  background: url("../assets/polygon.jpg") center center no-repeat;
  background-blend-mode: multiply;
  background-color: #484747;
  background-size: cover;
  color: #000;
  padding: 20px;
  margin-bottom: 40px;
  margin-left: 50px;
}

.favorite {
  display: flex;
  justify-content: flex-end;
}

.active-favorite {
  background: red;
}

.getWeather {
  margin: 40px 0;
}

.form input[type="text"] {
  background-color: transparent;
  border: 0;
  border-bottom: solid 2px #113a65;
  width: 50%;
  padding-bottom: 4px;
  color: #fff;
  font-weight: lighter;
  margin-right: 20px;
  font-size: 20px;
  outline: none;
}

.form input::-webkit-input-placeholder {
  color: #fff;
}

.form input::-moz-placeholder {
  color: #fff;
}

.form button:active {
  border-bottom: 2px solid transparent;
  border-color: transparent;
  position: relative;
  top: 2px;
}

.icon {
  margin-right: 15px;
}

.icon p {
  margin: 0;
  font-size: 16px;
}

.temp {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 15px 0;
  color: white;
  font-size: 20px;
  text-align: center;
}

.info-list p {
  color: white;
}

.buttons {
  display: flex;
  margin-top: 20px;
}

.line-chart {
  background-color: #fff;
  margin-top: 40px;
}

@media screen and (max-width: 1013px) {
  .infoWeath:first-child {
    margin-left: 0;
  }

  .form {
    display: flex;
    flex-direction: column;
  }

  .form input[type="text"] {
    width: 100%;
    margin-bottom: 15px;
  }

  .tabs {
    padding: 2px;
  }
}

@media screen and (max-width: 842px) {
  .infoWeath {
    width: 100%;
    margin-left: 0;
  }

  .form button {
    width: 60%;
    margin: auto;
  }

  .form input[type="text"] {
    width: 80%;
    margin: auto;
    margin-bottom: 15px;
  }

  .tabs {
    padding: 2px;
  }
}</style>