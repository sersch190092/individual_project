<template>
    <div>
      <h1>Wetter-App</h1>
      <input class="input_city" v-model="location" placeholder="Standort eingeben">
      <button class="button1" @click="getWeather">Wetter abrufen</button>
      <div v-if="weather">
        <h2>{{ weather.name }}</h2>
        <p>{{ weather.weather[0].description }}</p>
        <p>Temperatur: {{ weather.main.temp }}°C</p>
        <p>Luftfeuchtigkeit: {{ weather.main.humidity }}%</p>
      </div>
      <div v-if="error">{{ error }}</div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        location: '',
        weather: null,
        error: ''
      };
    },
    methods: {
      async getWeather() {
        try {
          const apiKey = process.env.VUE_APP_WEATHER_API_KEY;
          const response = await fetch(`http://api.openweathermap.org/geo/1.0/direct?q=${this.location}&limit=5&appid=${apiKey}`);
          if (!response.ok) {
            throw new Error('Standort nicht gefunden');
          }
          const data = await response.json();
          const { lat, lon } = data[0];
          const weatherResponse = await fetch(`http://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${apiKey}&units=metric`);
          if (!weatherResponse.ok) {
            throw new Error('Wetterdaten konnten nicht abgerufen werden');
          }
          this.weather = await weatherResponse.json();
          this.error = '';
        } catch (error) {
          this.weather = null;
          this.error = error.message;
        }
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
  </style>
  