<template>
  <div class="wrapper" :class="{ 'dark-mode': isDarkMode }">
    <div class="theme-toggle">
      <input
        type="checkbox"
        id="theme-toggle"
        v-model="isDarkMode"
        @change="toggleTheme"
      />
      <label for="theme-toggle" class="toggle-label">
        <span class="toggle-text">Dark Mode</span>
        <span class="toggle-slider"></span>
      </label>
    </div>
    <SearchComponent />
    <Clock></Clock>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';
import SearchComponent from './components/SearchComponent.vue';
import Clock from './components/Clock.vue';

const isDarkMode = ref(false);

function toggleTheme() {
  document.body.classList.toggle('dark-mode', isDarkMode.value);
}

watch(isDarkMode, () => {
  toggleTheme();
});
</script>

<style scoped>
.wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  transition: background-color 0.3s ease;
  font-family: 'Geologica', sans-serif;
  -webkit-text-stroke: 0.5px black;
}

.dark-mode {
  background-color: #222;
  color: #fff;
}

.wrapper {
  background-image: url('./assets/Day.jpg');
  background-size: cover;
  background-position: center;
  height: 98vh;
}

.wrapper.dark-mode {
  background-image: url('./assets/Night.jpg');
  background-size: cover;
  background-position: center;
}

.theme-toggle {
  position: absolute;
  top: 20px;
  right: 20px;
}

.toggle-label {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.toggle-text {
  margin-right: 10px;
}

.toggle-slider {
  display: inline-block;
  width: 40px;
  height: 20px;
  background-color: #ccc;
  border-radius: 10px;
  position: relative;
  transition: background-color 0.3s ease;
}

.toggle-slider:before {
  content: '';
  position: absolute;
  width: 16px;
  height: 16px;
  background-color: #fff;
  border-radius: 50%;
  top: 2px;
  left: 2px;
  transition: transform 0.3s ease;
}

#theme-toggle:checked + .toggle-label .toggle-slider {
  background-color: #00162e;
}

#theme-toggle:checked + .toggle-label .toggle-slider:before {
  transform: translateX(20px);
}

.weather-app {
  text-align: center;
  background-color: #fff;
  font-weight: bold;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  max-width: 400px;
  transition: background-color 0.3s ease, color 0.3s ease;
}

.dark-mode .weather-app {
  background-color: #aaa;
  color: #000;
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
  background-color: #6888aa;
}

.weather-results {
  background-color: #f5f5f5;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  text-align: left;
  max-width: 400px;
  margin: 0 auto;
  transition: background-color 0.3s ease, color 0.3s ease;
}

input[type='checkbox'] {
  display: none;
}

.dark-mode .weather-results {
  background-color: #444;
  color: #000;
}

h1 {
  color: #aaa;
  font-size: 24px;
  margin-bottom: 10px;
  transition: color 0.3s ease;
}

.dark-mode h1 {
  color: #aaa;
}

h2 {
  color: #555;
  margin-top: 0;
  font-size: 18px;
  margin-bottom: 10px;
}

.dark-mode h2 {
  color: #aaa;
}

p {
  margin-bottom: 5px;
}

.weather-results p:first-child {
  margin-top: 10px;
}

.dark-mode .weather-results p:first-child {
  margin-top: 10px;
  color: #ccc;
}

canvas {
  width: 100%;
  height: auto;
}
</style>
