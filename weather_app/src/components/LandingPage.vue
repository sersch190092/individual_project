<template>
  <div :class="{ 'dark-mode': isDarkMode }">
    <div class="toggle-dark-mode" @click="$emit('toggleDarkMode')">
      <span v-if="isDarkMode">🌞</span>
      <span v-else>🌙</span>
    </div>
    <h1>Weather-App</h1>
    <input class="input_city" v-model="cityInput" placeholder="City">
    <div class="button-container">
      <button class="button1" @click="showWeather">Show weather</button>
      <button class="button1" @click="showForecast">Forecast</button>
      <button class="button1" @click="showDetails">Details</button>
      <button class="button1" @click="getSimpleForecast">Forecast Simple</button>
    </div>

    <!-- Wetterdaten anzeigen -->
    <div v-if="mode === 'weather'">
      <div v-if="weatherData">
        <h2>{{ city }}</h2>
        <p>Temperature: {{ formatTemperature(weatherData.main.temp) }}°C</p>
        <p>Weather: {{ weatherData.weather[0].description }}</p>
      </div>
      <div v-else-if="errorMessage">
        <p>{{ errorMessage }}</p>
      </div>
    </div>

    <!-- Forecast anzeigen -->
    <div v-else-if="mode === 'forecast'">
      <div v-if="forecastData">
        <h2>Forecast for {{ city }}</h2>
        <div class="forecast-days">
          <div v-for="(day, index) in forecastData" :key="index" class="forecast-day">

            <h3>{{ day.date }}</h3>
            <div class="forecast-times_simple">
              <div v-for="(time, timeIndex) in day.times" :key="timeIndex" class="forecast-time">
                <p>{{ time.time }}</p>
                <p>Temperature: {{ formatTemperature(time.temp) }}°C</p>
                <p>Weather: {{ time.weather }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div v-else-if="errorMessage">
        <p>{{ errorMessage }}</p>
      </div>
    </div>

    <!-- Details anzeigen -->
    <div v-else-if="mode === 'details'">
      <h2>Details for {{ city }}</h2>
      <p>Latitude: {{ detailsData.lat }}</p>
      <p>Longitude: {{ detailsData.lon }}</p>
      <p>Country: {{ detailsData.country }}</p>
      <p>State: {{ detailsData.state }}</p>
    </div>

    <!-- Einfacher Forecast anzeigen -->
    <div v-else-if="mode === 'simpleForecast'">
      <div v-if="simpleForecastData">
        <h2 style="text-align: center">Simple Forecast for {{ city }}</h2>
        <div class="forecast-days_simple">
          <div v-for="(day, index) in simpleForecastData" :key="index" class="forecast-day_simple">
            <h3>{{ day.date }}</h3>
            <div class="forecast-times_simple">
              <p style="text-align: center">Minimum Temperature: {{ formatTemperature(day.minTemp) }}°C</p>
              <p style="text-align: center">Maximum Temperature: {{ formatTemperature(day.maxTemp) }}°C</p>
              <p style="text-align: center">Average Temperature: {{ formatTemperature(day.averageTemp) }}°C</p>
            </div>
          </div>
        </div>
      </div>
      <div v-else-if="errorMessage">
        <p>{{ errorMessage }}</p>
      </div>
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
      forecastData: null,
      detailsData: { lat: 0, lon: 0, country: "", state: "" },
      simpleForecastData: null,
      errorMessage: '',
      mode: ''
    };
  },
  methods: {
    showWeather() {
      this.mode = 'weather';
      this.getWeather();
    },
    showForecast() {
      this.mode = 'forecast';
      this.getForecast();
    },
    showDetails() {
      this.mode = 'details';
      this.getGeocodeDetails();
    },
    getWeather() {
      this.city = this.cityInput;
      const apiKey = process.env.VUE_APP_WEATHER_API_KEY;
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
    },
    getForecast() {
      this.city = this.cityInput;
      const apiKey = process.env.VUE_APP_WEATHER_API_KEY;
      const apiUrl = `https://api.openweathermap.org/data/2.5/forecast?q=${this.city}&appid=${apiKey}&units=metric`;

      axios.get(apiUrl)
        .then(response => {
          const data = response.data;
          this.forecastData = [];

          const today = new Date().toISOString().split('T')[0];

          const groupedData = {};
          data.list.forEach(item => {
            const date = item.dt_txt.split(' ')[0];
            if (date !== today) {
              if (!groupedData[date]) {
                groupedData[date] = [];
              }
              groupedData[date].push({
                time: item.dt_txt.split(' ')[1],
                temp: item.main.temp,
                weather: item.weather[0].description
              });
            }
          });

          for (const date in groupedData) {
            this.forecastData.push({ date, times: groupedData[date] });
          }

          this.errorMessage = '';
        })
        .catch(error => {
          this.forecastData = null;
          this.errorMessage = 'No forecast data available. Check if name is written correctly.';
          console.error('Error fetching forecast data:', error);
        });
    },
    getGeocodeDetails() {
      this.city = this.cityInput;
      const apiKey = process.env.VUE_APP_WEATHER_API_KEY;
      const apiUrl = `https://api.openweathermap.org/geo/1.0/direct?q=${this.city}&limit=1&appid=${apiKey}`;

      axios.get(apiUrl)
        .then(response => {
          const data = response.data[0];
          this.detailsData = {
            lat: data.lat,
            lon: data.lon,
            country: data.country,
            state: data.state
          };
          this.errorMessage = '';
        })
        .catch(error => {
          this.detailsData = null;
          this.errorMessage = 'No details available. Check if name is written correctly.';
          console.error('Error fetching geocode details:', error);
        });
    },
    getSimpleForecast() {
      this.mode = 'simpleForecast';
      this.calculateSimpleForecast();
    },
    calculateSimpleForecast() {
      if (!this.forecastData) return;

      this.simpleForecastData = [];
      this.forecastData.forEach(day => {
        let totalTemp = 0;
        let maxTemp = Number.MIN_SAFE_INTEGER;
        let minTemp = Number.MAX_SAFE_INTEGER;
        day.times.forEach(time => {
          totalTemp += time.temp;
          maxTemp = Math.max(maxTemp, time.temp);
          minTemp = Math.min(minTemp, time.temp);
        });
        const averageTemp = totalTemp / day.times.length;
        this.simpleForecastData.push({ date: day.date, maxTemp, minTemp, averageTemp });
      });
    },
    formatTemperature(temp) {
      return temp.toFixed(1); // Formatieren auf eine Nachkommastelle
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
  margin-right: 2px;
  margin-left: 2px;
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

.forecast-day {
  margin-bottom: 20px;
}

.forecast-days_simple {
  display: flex;
  justify-content: center;
}


.forecast-time {
  display: flex;
  flex-direction: column;
  padding: 20px;
}

.forecast-times_simple {
  display: flex;
  justify-content: center;
  margin: 10px 0px 10px 0px;
}

.button-container {
  display: flex;
  justify-content: center;
  margin-bottom: 10px;
}

.input_city,
.button1 {
  margin-bottom: 10px;
}

.forecast-day p {
  text-align: center;
}
</style>
