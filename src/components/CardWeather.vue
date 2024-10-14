<template>
  <div class="infoWeath" @click="$emit('getIdCard')">
    <div class="favorite">
      <button class="btn" @click="$emit('removeCard')">Видалити</button>
      <button class="btn" :class="{ 'active-favorite': showFavorite }" @click="$emit('addFavorite')">В Обране</button>
    </div>
    <div class="getWeather">

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
        <button class="btn" @click="$emit('getForFiveDays')">на 5 днів</button>
      </div>

      <div class="line-chart">
        <Bar :chart-data="chartData" />
      </div>

    </div>
  </div>
</template>

<script>
import { Bar } from 'vue-chartjs'
import { Chart, Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale } from 'chart.js'
import api from '../api/api.json'

Chart.register(Title, Tooltip, Legend, BarElement, CategoryScale, LinearScale)

export default {
  components: { Bar },
  props: {
    weather: {
      type: Object,
      default: () => {}
    },
    showFavorite: {
      type: Boolean,
      default: true
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
  max-width: 320px;
  height: auto;
  background: url("../assets/bg.jpeg") center center no-repeat;
  background-blend-mode: multiply;
  background-color: #484747;
  background-size: cover;
  color: #000;
  padding: 20px;
  margin-bottom: 40px;
  margin-left: 25px;
}

.favorite {
  display: flex;
  justify-content: flex-end;
}

.active-favorite {
  background: red;
}

.form input[type="text"] {
  background-color: transparent;
  border: 0;
  border-bottom: solid 2px #113a65;
  width: 100%;
  padding-bottom: 4px;
  color: #fff;
  font-weight: lighter;
  margin-bottom: 10px;
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
}
</style>