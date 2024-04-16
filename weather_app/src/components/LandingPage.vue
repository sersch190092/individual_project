<template>
  <div :class="{ 'dark-mode': isDarkMode }">
    <div class="toggle-dark-mode" @click="$emit('toggleDarkMode')">
      <span v-if="isDarkMode">🌞</span>
      <span v-else>🌙</span>
    </div>
    <h1>Weather-App</h1>
    <input class="input_city" v-model="cityInput" placeholder="City">
    <button class="button1" @click="getWeather">Show weather</button>
    <div v-if="weatherData">
      <h2>{{ city }}</h2>
      <p>Temperature: {{ weatherData.main.temp }}°C</p>
      <p>Weather: {{ weatherData.weather[0].description }}</p>
    </div>
    <div v-if="errorMessage">
      <p>{{ errorMessage }}</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  props: {
    isDarkMode: {
      type: Boolean,
      required: true
    }
  },
  data() {
    return {
      cityInput: '',
      city: '',
      weatherData: null,
      errorMessage: ''
    };
  },
  methods: {
    getWeather() {
      this.city = this.cityInput;
      const apiKey = process.env.WEATHER_API_KEY;
      console.log()
      const apiUrl = `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&appid=${apiKey}&units=metric`;

      axios.get(apiUrl)
        .then(response => {
          this.weatherData = response.data;
          this.errorMessage = '';
        })
        .catch(error => {
          this.weatherData = null;
          this.errorMessage = 'No data. Check if name is written correctly.';
          console.error('Error fetching weather data:', error);
        });
    }
  }
};
</script>

<style>
.input_city {
  border-radius: 5px;
  margin-right: 2px;
}

.button1 {
  border-radius: 5px;
}

.toggle-dark-mode {
  position: absolute;
  top: 10px;
  right: 10px;
  cursor: pointer;
}

.dark-mode {
  background-color: #333;
  color: #fff;
}
</style>
