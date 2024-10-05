<template>
  <div class="min-h-screen w-full bg-[url('../assets/360_F_641918542_bL3O5qWqKmaVrxM12Qa1pp1owFvKKP3k.jpg')] bg-cover bg-center bg-no-repeat flex items-center justify-center p-4 sm:p-8">
    <div class="w-full max-w-4xl grid grid-cols-1 md:grid-cols-3 gap-4 md:gap-6">
      <!-- Today's weather -->
      <div 
        v-if="currentWeather"
        class="md:col-span-1 rounded-3xl p-4 sm:p-6 text-center shadow-lg"
        :class="weatherColorClasses"
      >
      <h2 class="text-lg sm:text-xl font-semibold mb-2 sm:mb-4" :class="textColorClass">{{ location}}</h2>
        <h5 class="text-lg sm:text-xl font-semibold mb-2 sm:mb-4" :class="textColorClass">Today</h5>
        <div class="text-6xl sm:text-8xl font-bold flex items-center justify-center mb-2 sm:mb-4" :class="textColorClass">
          <i :class="['text-5xl sm:text-6xl mr-2 sm:mr-4', getWeatherIcon(currentWeather.weather[0]?.main)]"></i>
          {{ Math.round(currentWeather.main?.temp) }}°
        </div>
        <p class="text-xl mt-2" :class="textColorClass">{{ formattedWeatherDescription }}</p>
        <p class="text-sm mt-1" :class="secondaryTextColorClass">
          Feels like {{ Math.round(currentWeather.main?.feels_like) }}° • 
          Sunset {{ formatTime(currentWeather.sys?.sunset) }}
        </p>
      </div>

      <!-- Loading state -->
      <div v-else class="md:col-span-1 rounded-3xl p-4 sm:p-6 text-center shadow-lg bg-gray-200">
        <p class="text-gray-600">Loading weather data...</p>
      </div>

      <!-- 5-day forecast -->
      <div class="md:col-span-2 bg-white bg-opacity-90 rounded-3xl p-4 sm:p-6 shadow-lg">
        <h3 class="text-lg sm:text-xl font-semibold text-gray-700 mb-4">5-Day Forecast for {{ location }}</h3>
        <div v-if="forecast.length" class="grid grid-cols-5 gap-2 sm:gap-4 text-center">
          <div v-for="day in forecast" :key="day.dt" class="flex flex-col items-center">
            <p class="text-xs sm:text-sm font-medium text-gray-600 mb-2">{{ formatDate(day.dt) }}</p>
            <div class="bg-gray-100 rounded-full p-2 mb-2">
              <i :class="['text-2xl sm:text-3xl text-gray-700', getWeatherIcon(day.weather[0]?.main)]"></i>
            </div>
            <p class="text-sm sm:text-base font-bold text-gray-800">{{ Math.round(day.main?.temp) }}°</p>
            <p class="text-xs sm:text-sm text-gray-600 mt-1">{{ day.weather[0]?.main }}</p>
          </div>
        </div>
        <p v-else class="text-gray-600 text-center">Loading forecast data...</p>
      </div>

      <div class="md:col-span-3 bg-gray-100 bg-opacity-80 rounded-3xl p-4 sm:p-6 shadow-lg">
        <h3 class="text-lg sm:text-xl font-semibold text-gray-700 mb-2 sm:mb-3">Weather Summary</h3>
        <p v-if="currentWeather" class="text-sm sm:text-base text-gray-600 mb-4">
          {{ weatherSummary }}
        </p>
        <h4 class="text-md sm:text-lg font-semibold text-gray-700 mb-2">5-Day Forecast Overview</h4>
        <p v-if="forecast.length" class="text-sm sm:text-base text-gray-600">
          {{ forecastSummary }}
        </p>
        <p v-else class="text-sm sm:text-base text-gray-600">
          Loading forecast data...
        </p>
      </div>

      <!-- Random text -->
      
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const API_KEY = '';
    const currentWeather = ref(null);
    const forecast = ref([]);
    const searchQuery = ref('');
    const error = ref('');
    const location = ref('');

    const fetchWeather = async (city) => {
      try {
        const currentResponse = await axios.get(`https://api.openweathermap.org/data/2.5/weather?q=${city}&units=metric&appid=${API_KEY}`);
        currentWeather.value = currentResponse.data;
        location.value = currentResponse.data.name;
        const forecastResponse = await axios.get(`https://api.openweathermap.org/data/2.5/forecast?q=${city}&units=metric&appid=${API_KEY}`);
        forecast.value = forecastResponse.data.list.filter((reading, index) => index % 8 === 0);

        error.value = '';
      } catch (err) {
        error.value = 'Failed to fetch weather data. Please try again.';
        console.error(err);
      }
    };

    const searchLocation = () => {
      if (searchQuery.value.trim()) {
        fetchWeather(searchQuery.value);
      }
    };

    const formatDate = (timestamp) => {
      const date = new Date(timestamp * 1000);
      return date.toLocaleDateString('en-US', { weekday: 'short' });
    };

    const formatTime = (timestamp) => {
      if (!timestamp) return 'N/A';
      const date = new Date(timestamp * 1000);
      return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    };

    const getWeatherIcon = (condition) => {
      if (!condition) return 'fas fa-question';
      switch (condition.toLowerCase()) {
        case 'clear': return 'fas fa-sun';
        case 'haze': return 'fas fa-sun';
        case 'clouds': return 'fas fa-cloud';
        case 'rain': return 'fas fa-cloud-rain';
        case 'drizzle': return 'fas fa-cloud-rain';
        case 'thunderstorm': return 'fas fa-bolt';
        case 'snow': return 'fas fa-snowflake';
        case 'mist':
        case 'fog': return 'fas fa-smog';
        default: return 'fas fa-question';
      }
    };

    const weatherColorClasses = computed(() => {
      const weatherMain = currentWeather.value?.weather[0]?.main?.toLowerCase() || '';
      switch(weatherMain) {
        case 'clear': return 'bg-orange-300';
        case 'haze': return 'bg-orange-300';
        case 'clouds': return 'bg-gray-400';
        case 'rain':
        case 'drizzle': return 'bg-blue-600';
        case 'thunderstorm': return 'bg-purple-600';
        case 'snow': return 'bg-blue-200';
        case 'mist':
        case 'fog': return 'bg-gray-300';
        default: return 'bg-orange-100';
      }
    });

    const formattedWeatherDescription = computed(() => {
      const description = currentWeather.value?.weather[0]?.description || '';
      return description.charAt(0).toUpperCase() + description.slice(1);
    });

    const textColorClass = computed(() => {
      const weatherMain = currentWeather.value?.weather[0]?.main?.toLowerCase() || '';
      return ['rain', 'drizzle', 'thunderstorm'].includes(weatherMain) ? 'text-white' : 'text-gray-800';
    });

    const secondaryTextColorClass = computed(() => {
      const weatherMain = currentWeather.value?.weather[0]?.main?.toLowerCase() || '';
      return ['rain', 'drizzle', 'thunderstorm'].includes(weatherMain) ? 'text-gray-200' : 'text-gray-600';
    });

    const weatherSummary = computed(() => {
      if (!currentWeather.value) return '';
      const { main, weather, wind } = currentWeather.value;
      return `Currently in ${currentWeather.value.name}, it's ${Math.round(main.temp)}°C with ${weather[0].description}. 
      The humidity is ${main.humidity}% and wind speed is ${wind.speed} m/s. 
      Today's high will be ${Math.round(main.temp_max)}°C and the low will be ${Math.round(main.temp_min)}°C.`;
    });

    const forecastSummary = computed(() => {
      if (forecast.value.length === 0) return '';
      let summary = 'Over the next 5 days: ';
      forecast.value.forEach((day, index) => {
        summary += `${formatDate(day.dt)}: ${Math.round(day.main.temp)}°C, ${day.weather[0].main}`;
        if (index < forecast.value.length - 1) summary += ' | ';
      });
      return summary;
    });

    onMounted(() => {
      fetchWeather('delhi'); // Default city
    });

    return {
      currentWeather,
      forecast,
      searchQuery,
      error,
      searchLocation,
      formatDate,
      formatTime,
      getWeatherIcon,
      weatherColorClasses,
      formattedWeatherDescription,
      textColorClass,
      secondaryTextColorClass,
      weatherSummary,
      forecastSummary,
      location
    };
  }
};
</script>

<style scoped>
/* Add any additional styles if necessary */
</style>
      