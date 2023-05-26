<template>
  <div id="app">
     <h1>Weather App</h1>
 

     <div class="recent-locations">
      <button v-for="(location, index) in Array.isArray(savedLocations) ? savedLocations.slice(0, 3) : []" :key="index" @click="getWeather(location)" class="location-button">
        {{ location }}
      </button>
    </div>

    <div v-if="savedLocations">
      <h2>{{ savedLocations.name }}</h2>
      <p>{{ savedLocations.main.temp }}°C</p>
      <p>{{ savedLocations.weather[0].description }}</p>
      <img :src="getWeatherIconURL(savedLocations.weather[0].icon)" :alt="savedLocations.weather[0].description">
      <p>Humidity: {{ savedLocations.main.humidity }}%</p>
      <p>Wind Speed: {{ savedLocations.wind.speed }} m/s</p>
      <p>Pressure: {{ savedLocations.main.pressure }} hPa</p>
    </div>

     
     <div class="search-bar">
       <input type="text" v-model="searchQuery" placeholder="Enter location">
       <button @click="searchWeather">Search</button>
     </div>
 
        <div class="weather-info"> 
     
       <div v-if="weatherData">
         <div class="current-weather">
           <h2>{{ weatherData.name }}</h2>
           <p>{{ weatherData.main.temp }}°C</p>
           <p>{{ weatherData.weather[0].description }}</p>
           <img :src="getWeatherIconURL(weatherData.weather[0].icon)" :alt="weatherData.weather[0].description">
           <p>Humidity: {{ weatherData.main.humidity }}%</p>
           <p>Wind Speed: {{ weatherData.wind.speed }} m/s</p>
           <p>Pressure: {{ weatherData.main.pressure }} hPa</p>
           <p>Date and Time: {{ getFormattedDateTime() }}</p>
         </div> 
 
         <div class="weather-forecast" v-if="weatherForecast.length">
           <h3>Weather Forecast</h3>
           <ul>
             <li v-for="forecast in weatherForecast" :key="forecast.dt">
               <p>{{ getFormattedTime(forecast.dt) }}</p>
               <p>{{ forecast.main.temp }}°C</p>
               <img :src="getWeatherIconURL(forecast.weather[0].icon)" :alt="forecast.weather[0].description">
             </li>
           </ul>
         </div> 
        </div> 

         </div> 
     </div>
 </template>  

<script setup lang="ts">
import axios from 'axios';
import { ref, getCurrentInstance } from 'vue';


interface WeatherData {
  name: string;
  main: {
    temp: number;
    humidity: number;
    pressure: number;
  };
  weather: {
    description: string;
    icon: string;
  }[];
  wind: {
    speed: number;
  };
}


interface Forecast {
  dt: number;
  main: {
    temp: number;
  };
  weather: {
    description: string;
    icon: string;
  }[];
}


const savedLocations = ref<WeatherData | null>(null);
const searchQuery = ref('');
const weatherData = ref<WeatherData | null>(null);
const searchWeather = ref();
const weatherForecast = ref<Forecast[]>([]);
const getFormattedDateTime = ref<string>('');
const getFormattedTime = ref<string>('');
const getWeatherIconURL = ref<string>('');
const getWeather = ref();


export default {
  name: 'App',
  setup() {
    const savedLocations = ref([]);
    const savedLocationsFromStorage = localStorage.getItem('savedLocations');
    if (savedLocationsFromStorage) {
      savedLocations.value = JSON.parse(savedLocationsFromStorage);
    }
   

  return {
      savedLocations,
      searchQuery,
      weatherData,
      weatherForecast,
      searchWeather,
      getFormattedDateTime,
      getFormattedTime,
      getWeatherIconURL,
      getWeather
  }
}
    

  methods: {
    getWeather(location: string): void {
      this.fetchWeatherData(location),
      this.storeLocation(location);
    },
  

    searchWeather(): void {
      if (this.searchQuery.trim() === '') {
        return;
      }
      this.fetchWeatherData(this.searchQuery); // Fetches weather data for the searched location
      
      this.searchQuery = ''; // Clears the search input

       this.storeLocation(this.searchQuery); // Stores the searched location in localStorage
    },


    storeLocation(location: string): void {
      if (this.savedLocations.length === 3) {   // Removes the oldest location if there are already three saved locations
        this.savedLocations.pop();
      }
      
      this.savedLocations.unshift(location); // Adds the new location to the beginning of the array

        
        localStorage.setItem('savedLocations', JSON.stringify(this.savedLocations)); 
    },


    getWeatherIconURL(icon: string): string {
      const apiKey = '26e133a61bfa8cd132ef2f59ede3bcb2'; 
      return `https://openweathermap.org/img/w/${icon}.png?appid=${apiKey}`;
    },


    getFormattedDateTime(): string {
      const date = new Date();
      const options: Intl.DateTimeFormatOptions = {
      weekday: 'long',
      year: 'numeric',
      month: 'long',
      day: 'numeric',
      hour: 'numeric',
      minute: 'numeric',
      second: 'numeric',
      timeZoneName: 'short'
    };
      return date.toLocaleString('en-US', options);
    },

    getFormattedTime(dt: number): string {
      const date = new Date(dt * 1000);
      return date.toLocaleTimeString(navigator.language, {
        hour: '2-digit',
        minute: '2-digit',
      });
    },

  
     async fetchWeatherData(location: string): Promise <void> {
      const apiKey = '26e133a61bfa8cd132ef2f59ede3bcb2'; 
      const url = `https://api.openweathermap.org/data/2.5/weather?q=${location}&appid=${apiKey}&units=metric`;
      const response = await axios.get(url);
      this.weatherData = response.data;
     },

 
      async fetchWeatherForecast(location: string): Promise<void> {
      const apiKey = '26e133a61bfa8cd132ef2f59ede3bcb2';
      const forecastUrl = `https://api.openweathermap.org/data/2.5/forecast?q=${location}&cnt=3&appid=${apiKey}&units=metric`;
      const forecastResponse = await axios.get(forecastUrl);
      this.weatherForecast = forecastResponse.data.list;
    }
  }
}
</script>



<style>
#app {
  text-align: center;
}

h1 {
  margin-top: 20px;
  font-size: 24px;
  font-weight: bold;
}

.recent-locations {
  margin-top: 20px;
}

.location-button {
  display: block;
  margin-bottom: 10px;
  padding: 10px;
  background-color: #eaeaea;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.location-button:hover {
  background-color: #d5d5d5;
}

.search-bar {
  margin: 20px 0;
}

input {
  padding: 5px;
  margin-right: 10px;
}

button {
  padding: 5px 10px;
}

.weather-info {
  margin-top: 20px;
}

.current-weather {
  margin-bottom: 20px;
}

.current-weather img {
  width: 50px;
  height: 50px;
}

.weather-forecast {
  margin-top: 20px;
}

.weather-forecast h3 {
  margin-bottom: 10px;
}

.weather-forecast ul {
  list-style: none;
  padding: 0;
}

.weather-forecast li {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.weather-forecast li img {
  width: 30px;
  height: 30px;
  margin-right: 10px;
}
</style>
