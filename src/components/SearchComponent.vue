<template>
  <div class="weather-app" :style="{ 'background-color': weatherBackgroundColor }">
    <h1 class="white">Weather App</h1>
    <div class="weather-search">
      <label for="location-input" class="white">Enter a location:</label>
      <input
        type="text"
        id="location-input"
        v-model="location"
        @keyup.enter="fetchWeatherData"
      />
      <button @click="fetchWeatherData">Search</button>
    </div>
    <div
      class="weather-results"
      v-if="weatherData"
      :style="{ 'background-image': weatherBackground }"
    >
      <h2 class="white">Current Weather in {{ weatherData.city }}</h2>
      <div class="temperatures">
        <p class="temperature">{{ weatherData.temperature }}°C</p>
        <div>
          <span v-if="weatherData.conditions === 'clear sky'">
            <img src="../assets/icons/sun.png" alt="Sun" class="weather-icon" />
          </span>
          <span v-else-if="isCloudyCondition(weatherData.conditions)">
            <img src="../assets/icons/cloud.png" alt="Cloud" class="weather-icon" />
          </span>
          <span v-else-if="isRainyCondition(weatherData.conditions)">
            <img src="../assets/icons/rain.png" alt="Rain" class="weather-icon" />
          </span>
        </div>
      </div>
      <p class="white">{{ capitalized(weatherData.conditions) }}</p>
      <p class="white">Wind Speed: {{ weatherData.windSpeed }} m/s</p>

      <h2 class="white">Weekly Temperature Chart</h2>
      <canvas ref="temperatureChart" width="400" height="300"></canvas>
    </div>
    <p v-if="error" class="white">{{ error }}</p>
  </div>
</template>

<script>
import Chart from 'chart.js/auto';
import day from '@/assets/day.jpg';
import broken_clouds from '@/assets/broken_clouds.jpg';
import heavy_intensity_rain from '@/assets/heavy_intensity_rain.webp';
import light_rain from '@/assets/light_rainnn.jpg';
import haze from '@/assets/haze.jpg';
import mist from '@/assets/what-is-mist.jpg';
import moderate_rain from '@/assets/moderate_rain_rain.jpg';
import overcast_clouds from '@/assets/overcast_clouds.jpg';
import scattered_clouds from '@/assets/scattered_clouds.jpg';
import thunderstorm from '@/assets/thunderstorm.jpg';
import very_heavy_rain from '@/assets/very_heavy_rainnn.jpg';
import few_clouds from '@/assets/few_clouds.avif';

function extractWeeklyTemperature(forecastData) {
  const weeklyTemperature = [];
  const forecastList = forecastData.list;

  let currentDate = null;
  let dailyTemperatures = [];

  for (const forecast of forecastList) {
    const forecastDateTime = new Date(forecast.dt * 1000);
    const forecastDate = forecastDateTime.toDateString();

    if (!currentDate) {
      currentDate = forecastDate;
    }

    if (forecastDate !== currentDate) {
      weeklyTemperature.push({
        date: currentDate,
        temperatures: dailyTemperatures,
      });
      currentDate = forecastDate;
      dailyTemperatures = [];
    }

    dailyTemperatures.push(Math.round(forecast.main.temp));
  }

  weeklyTemperature.push({
    date: currentDate,
    temperatures: dailyTemperatures,
  });

  return weeklyTemperature;
}

export default {
  data() {
    return {
      location: '',
      weatherData: null,
      error: '',
      weeklyTemperature: null,
      chartInstance: null,
    };
  },
  updated() {
    if (this.weeklyTemperature) {
      this.renderTemperatureChart();
    }
  },
  methods: {
    isCloudyCondition(condition) {
      return (
        condition === 'few clouds' ||
        condition === 'overcast clouds' ||
        condition === 'mist' ||
        condition === 'broken clouds' ||
        condition === 'scattered clouds' ||
        condition === 'haze'
      );
    },
    isRainyCondition(condition) {
      return (
        condition === 'heavy intensity rain' ||
        condition === 'light rain' ||
        condition === 'very heavy rain' ||
        condition === 'moderate rain' ||
        condition === 'Light intensity shower rain'
      );
    },
    capitalized(name) {
      const capitalizedFirst = name[0].toUpperCase();
      const rest = name.slice(1);

      return capitalizedFirst + rest;
    },
    async fetchWeatherData() {
      const apiKey = '38c9500969a9d3539fdf1086fe80d399';
      const currentWeatherEndpoint = 'https://api.openweathermap.org/data/2.5/weather';
      const forecastEndpoint = 'https://api.openweathermap.org/data/2.5/forecast';
      const currentWeatherUrl = new URL(currentWeatherEndpoint);
      const forecastUrl = new URL(forecastEndpoint);

      currentWeatherUrl.searchParams.append('q', this.location);
      currentWeatherUrl.searchParams.append('appid', apiKey);
      currentWeatherUrl.searchParams.append('units', 'metric');

      forecastUrl.searchParams.append('q', this.location);
      forecastUrl.searchParams.append('appid', apiKey);
      forecastUrl.searchParams.append('units', 'metric');

      this.destroyChartInstance();

      try {
        const currentWeatherResponse = await fetch(currentWeatherUrl.toString());
        const forecastResponse = await fetch(forecastUrl.toString());
        const currentWeatherData = await currentWeatherResponse.json();
        const forecastData = await forecastResponse.json();

        if (currentWeatherResponse.ok && forecastResponse.ok) {
          this.weatherData = {
            city: currentWeatherData.name,
            temperature: Math.round(currentWeatherData.main.temp),
            conditions: currentWeatherData.weather[0].description,
            windSpeed: currentWeatherData.wind.speed,
          };

          this.weeklyTemperature = extractWeeklyTemperature(forecastData);

          this.error = '';
        } else {
          this.error = 'Error fetching weather data: ' + currentWeatherData.message;
          console.error('Error fetching weather data:', currentWeatherData.message);
        }
      } catch (error) {
        this.error = 'Error fetching weather data: ' + error.message;
        console.error('Error fetching weather data:', error);
      }
    },
    renderTemperatureChart() {
      if (!this.weeklyTemperature) {
        return;
      }

      const canvas = this.$refs.temperatureChart;
      if (!canvas) {
        return;
      }

      this.$nextTick(() => {
        const ctx = canvas.getContext('2d');
        const dates = this.weeklyTemperature.map((day) => day.date);
        const temperatures = this.weeklyTemperature.map((day) => day.temperatures);

        if (!this.chartInstance) {
          this.chartInstance = new Chart(ctx, {
            type: 'line',
            data: {
              labels: dates,
              datasets: [
                {
                  label: 'Temperature (°C)',
                  data: temperatures.flat(),
                  borderColor: 'rgb(75, 192, 192)',
                  tension: 0.4,
                },
              ],
            },
            options: {
              scales: {
                y: {
                  beginAtZero: true,
                  title: {
                    display: true,
                    text: 'Temperature (°C)',
                  },
                },
              },
            },
          });
        } else {
          this.chartInstance.data.labels = dates;
          this.chartInstance.data.datasets[0].data = temperatures.flat();
          this.chartInstance.update();
        }
      });
    },
    destroyChartInstance() {
      if (this.chartInstance) {
        this.chartInstance.destroy();
        this.chartInstance = null;
      }
    },
  },
  computed: {
    weatherBackground() {
      if (this.weatherData && this.weatherData.conditions === 'clear sky') {
        return `url(${day})`;
      }
      if (this.weatherData && this.weatherData.conditions === 'few clouds') {
        return `url(${few_clouds})`;
      }
      if (this.weatherData && this.weatherData.conditions === 'overcast clouds') {
        return `url(${overcast_clouds})`;
      }
      if (this.weatherData && this.weatherData.conditions === 'heavy intensity rain') {
        return `url(${heavy_intensity_rain})`;
      }
      if (this.weatherData && this.weatherData.conditions === 'mist') {
        return `url(${mist})`;
      }
      if (this.weatherData && this.weatherData.conditions === 'broken clouds') {
        return `url(${broken_clouds})`;
      }
      if (this.weatherData && this.weatherData.conditions === 'thunderstorm') {
        return `url(${thunderstorm})`;
      }
      if (this.weatherData && this.weatherData.conditions === 'light rain') {
        return `url(${light_rain})`;
      }
      if (this.weatherData && this.weatherData.conditions === 'very heavy rain') {
        return `url(${very_heavy_rain})`;
      }
      if (
        (this.weatherData && this.weatherData.conditions === 'moderate rain') ||
        'Light intensity shower rain'
      ) {
        return `url(${moderate_rain})`;
      }
      if (this.weatherData && this.weatherData.conditions === 'scattered clouds') {
        return `url(${scattered_clouds})`;
      }
      if (this.weatherData && this.weatherData.conditions === 'haze') {
        return `url(${haze})`;
      }
      return '';
    },
    weatherBackgroundColor() {
      if (this.weatherData) {
        const conditions = this.weatherData.conditions.toLowerCase();

        if (conditions.includes('clear sky')) {
          return 'rgba(0, 0, 255, 0.1)';
        } else if (
          conditions.includes(
            'few clouds' ||
              'overcast clouds' ||
              'mist' ||
              'broken clouds' ||
              'scattered clouds' ||
              'haze'
          )
        ) {
          return 'rgba(128, 128, 128, 0.1)';
        } else if (
          conditions.includes(
            'heavy intensity rain' ||
              'light rain' ||
              'very heavy rain' ||
              'moderate rain' ||
              'Light intensity shower rain'
          )
        ) {
          return 'navy';
        } else if (conditions.includes('thunderstorm')) {
          return 'red';
        }
        return 'rgba(0, 0, 255, 0.1)';
      }
      return 'rgba(0, 0, 255, 0.1)';
    },
  },
  mounted() {
    this.renderTemperatureChart();
  },
};
</script>

<style>
.white {
  color: #fff;
}
.temperature {
  font-size: 40px;
  color: #fff;
}

.weather-app {
  text-align: center;
}

.weather-search {
  margin-bottom: 20px;
}

input[type='text'] {
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
  margin-right: 10px;
}

button {
  padding: 10px;
  border-radius: 5px;
  background-color: #007bff;
  color: #fff;
  border: none;
  cursor: pointer;
}

button:hover {
  background-color: #0062cc;
}

.weather-results {
  background-color: #f5f5f5;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  text-align: left;
  max-width: 400px;
  margin: 0 auto;
}

h1 {
  color: #333;
}

h2 {
  color: #555;
  margin-top: 0;
}

p {
  margin-bottom: 5px;
}

.weather-results p:first-child {
  margin-top: 10px;
}
.weather-icon {
  width: 50px;
  height: 50px;
  margin: 15px;
}
.temperatures {
  display: flex;
  justify-content: space-between;
}
</style>
