Yes! To make the search area fully functional and interactive, we can upgrade your location pill into an input box with a dropdown or search submit so you can query any city on the fly and fetch its live forecast using your API key.

Here is the updated Vue code featuring a **working search input** for the location pill, which triggers a live API fetch for Paris or any other city you type in:

```vue
<template>
  <main class="weather-app">
    <div class="weather-container">
      <!-- Header with Search Input -->
      <header class="topbar">
        <form @submit.prevent="handleSearch" class="location-pill-form">
          <div class="location-pill">
            <span>📍</span>
            <input 
              type="text" 
              v-model="searchQuery" 
              placeholder="Search city..." 
              class="city-input"
            />
            <button type="submit" class="search-btn">🔍</button>
          </div>
        </form>

        <button class="profile-button" aria-label="Profile"></button>    
      </header>

      <!-- Page heading -->
      <section class="page-heading">
        <div>
          <p class="eyebrow">YOUR WEATHER DASHBOARD</p>
          <h1>Good day<span class="heading-dot">.</span></h1>
          <p class="date-text">{{ today }} · Here's your forecast</p>
        </div>

        <div class="live-label">
          <span class="live-dot"></span>
          Weather overview
        </div>
      </section>

      <!-- Main weather card -->
      <section class="main-weather">
        <div class="weather-orb orb-one"></div>
        <div class="weather-orb orb-two"></div>

        <div class="main-weather-content">
          <div class="current-info">
            <span class="current-label">CURRENT WEATHER</span>
            <h2>{{ city }}</h2>
            <p class="current-date">{{ today }}</p>

            <div class="temperature-row">
              <span class="main-icon">{{ currentWeather.icon }}</span>
              <div class="temperature">
                {{ currentWeather.temperature }}<span>°</span>
              </div>
            </div>

            <p class="condition">{{ currentWeather.condition }}</p>

            <div class="high-low">
              <span>↑ {{ currentWeather.high }}° High</span>
              <span>↓ {{ currentWeather.low }}° Low</span>
            </div>
          </div>

          <div class="weather-details">
            <div class="detail-card">
              <span class="detail-icon">💧</span>
              <span class="detail-label">Humidity</span>
              <strong>{{ currentWeather.humidity }}%</strong>
              <span class="detail-note">Moisture in air</span>
            </div>

            <div class="detail-card">
              <span class="detail-icon">💨</span>
              <span class="detail-label">Wind speed</span>
              <strong>{{ currentWeather.wind }} <small>km/h</small></strong>
              <span class="detail-note">Gentle breeze</span>
            </div>

            <div class="detail-card">
              <span class="detail-icon">🌧️</span>
              <span class="detail-label">Chance of rain</span>
              <strong>{{ currentWeather.rain }}%</strong>
              <span class="detail-note">Precipitation</span>
            </div>
          </div>
        </div>

        <span class="weather-decoration decoration-one"></span>
        <span class="weather-decoration decoration-two"></span>
      </section>

      <!-- Hourly forecast -->
      <section class="section-block">
        <div class="section-heading">
          <div>
            <p class="eyebrow">PLAN YOUR DAY</p>
            <h2>Hourly forecast</h2>
          </div>
          <span class="section-caption">Next few hours</span>
        </div>

        <div class="hourly-list">
          <button
            v-for="(hour, index) in hourlyForecast"
            :key="hour.time"
            class="hour-card"
            :class="{ active: selectedHour === index }"
            @click="selectedHour = index"
          >
            <span class="hour-time">{{ hour.time }}</span>
            <span class="hour-icon">{{ hour.icon }}</span>
            <strong>{{ hour.temp }}°</strong>
          </button>
        </div>
      </section>

      <!-- Weekly forecast -->
      <section class="section-block weekly-section">
        <div class="section-heading">
          <div>
            <p class="eyebrow">LOOKING AHEAD</p>
            <h2>7-day forecast</h2>
          </div>
          <span class="section-caption">This week</span>
        </div>

        <div class="weekly-card">
          <div
            v-for="(day, index) in weeklyForecast"
            :key="day.day"
            class="day-row"
            :class="{ 'today-row': index === 0 }"
          >
            <div class="day-name">
              <strong>{{ day.day }}</strong>
              <span v-if="index === 0">Today</span>
            </div>

            <span class="day-icon">{{ day.icon }}</span>

            <span class="day-condition">{{ day.condition }}</span>

            <div class="temperature-range">
              <span class="low-temp">{{ day.low }}°</span>
              <div class="range-track">
                <span
                  class="range-fill"
                  :style="{
                    left: `${Math.max(0, (day.low - 20) * 5)}%`,
                    width: `${Math.max(12, (day.temp - day.low) * 5)}%`
                  }"
                ></span>
              </div>
              <strong>{{ day.temp }}°</strong>
            </div>
          </div>
        </div>
      </section>

      <footer class="footer">
        <span>Made for a brighter day.</span>
        <span class="demo-label">LIVE DATA</span>
      </footer>
    </div>
  </main>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const API_KEY = '330f6ab158734dcb96e05901262610'
const city = ref('Paris')
const searchQuery = ref('Paris')
const selectedHour = ref(0)

const currentWeather = ref({
  temperature: 0,
  condition: 'Loading...',
  high: 0,
  low: 0,
  humidity: 0,
  wind: 0,
  rain: 0,
  icon: '🌤️'
})

const hourlyForecast = ref([])
const weeklyForecast = ref([])

const getWeatherIcon = (code) => {
  if ([1000].includes(code)) return '☀️'
  if ([1003, 1006].includes(code)) return '🌤️'
  if ([1009, 1030, 1135, 1147].includes(code)) return '☁️'
  if ([1063, 1180, 1183, 1186, 1189, 1192, 1195, 1240].includes(code)) return '🌧️'
  if ([1087, 1273, 1276].includes(code)) return '⛈️'
  if ([1066, 1114, 1210, 1213, 1216, 1219, 1222, 1225].includes(code)) return '❄️'
  return '🌦️'
}

const fetchWeatherData = async (targetCity) => {
  try {
    const response = await fetch(
      `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=${encodeURIComponent(targetCity)}&days=7&aqi=no&alerts=no`
    )
    if (!response.ok) throw new Error('City not found')
    
    const data = await response.json()

    city.value = `${data.location.name}, ${data.location.country}`

    const current = data.current
    const todayData = data.forecast.forecastday[0]

    currentWeather.value = {
      temperature: Math.round(current.temp_c),
      condition: current.condition.text,
      high: Math.round(todayData.day.maxtemp_c),
      low: Math.round(todayData.day.mintemp_c),
      humidity: current.humidity,
      wind: Math.round(current.wind_kph),
      rain: todayData.day.daily_chance_of_rain,
      icon: getWeatherIcon(current.condition.code)
    }

    const currentHour = new Date(data.location.localtime).getHours()
    hourlyForecast.value = todayData.hour
      .slice(currentHour, currentHour + 6)
      .map((hourItem) => {
        const dateObj = new Date(hourItem.time)
        const hour12 = dateObj.toLocaleTimeString('en-US', { hour: 'numeric', hour12: true })
        return {
          time: hour12,
          icon: getWeatherIcon(hourItem.condition.code),
          temp: Math.round(hourItem.temp_c)
        }
      })

    weeklyForecast.value = data.forecast.forecastday.map((dayItem) => {
      const dateObj = new Date(dayItem.date)
      const dayName = dateObj.toLocaleDateString('en-US', { weekday: 'long' })
      return {
        day: dayName,
        icon: getWeatherIcon(dayItem.day.condition.code),
        temp: Math.round(dayItem.day.maxtemp_c),
        low: Math.round(dayItem.day.mintemp_c),
        condition: dayItem.day.condition.text
      }
    })
  } catch (error) {
    console.error('Error fetching weather data:', error)
    alert('Could not find weather for that city. Please try again.')
  }
}

const handleSearch = () => {
  if (searchQuery.value.trim()) {
    fetchWeatherData(searchQuery.value.trim())
  }
}

onMounted(() => {
  fetchWeatherData('Paris')
})

const today = new Intl.DateTimeFormat('en-US', {
  weekday: 'long',
  month: 'long',
  day: 'numeric'
}).format(new Date())
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&family=Manrope:wght@400;500;600;700;800&display=swap');

:root {
  font-family: 'DM Sans', sans-serif;
  color: #f5f7ff;
  background: #071020;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  min-width: 320px;
}

button {
  font: inherit;
}

.weather-app {
  min-height: 100vh;
  padding: 28px 24px;
  overflow: hidden;
  background:
    radial-gradient(ellipse at 15% 5%, #12294a 0, transparent 34%),
    radial-gradient(ellipse at 95% 45%, #102342 0, transparent 35%),
    #071020;
}

.weather-container {
  width: 100%;
  max-width: 1060px;
  margin: 0 auto;
}

/* Header */

.topbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 18px;
  margin-bottom: 56px;
}

.location-pill-form {
  display: flex;
  align-items: center;
}

.location-pill {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 6px 14px;
  border: 1px solid #283b5b;
  border-radius: 30px;
  background: #101d32;
  color: #e0e9fa;
  font-size: 13px;
}

.city-input {
  background: transparent;
  border: none;
  color: #ffffff;
  font-family: inherit;
  font-size: 13px;
  outline: none;
  width: 130px;
}

.city-input::placeholder {
  color: #91a6c5;
}

.search-btn {
  background: transparent;
  border: none;
  cursor: pointer;
  font-size: 14px;
  padding: 2px;
}

.profile-button {
  width: 43px;
  height: 43px;
  border: 1px solid #30405d;
  border-radius: 50%;
  background: #15233b;
  color: white;
  font-size: 23px;
  cursor: pointer;
}

/* Heading */

.page-heading {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 20px;
  margin-bottom: 26px;
}

.eyebrow {
  margin: 0 0 9px;
  color: #8ba9d4;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 2px;
}

.page-heading h1 {
  margin: 0;
  font-family: 'Manrope', sans-serif;
  font-size: clamp(30px, 4vw, 43px);
  font-weight: 800;
  letter-spacing: -2px;
}

.heading-dot {
  color: #73b5ff;
}

.date-text {
  margin: 8px 0 0;
  color: #9cabc4;
  font-size: 13px;
}

.live-label {
  display: flex;
  align-items: center;
  gap: 8px;
  padding-bottom: 5px;
  color: #bdcbe0;
  font-size: 12px;
}

.live-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #66e5bd;
  box-shadow: 0 0 12px #66e5bd88;
}

/* Main weather card */

.main-weather {
  position: relative;
  isolation: isolate;
  overflow: hidden;
  padding: 34px;
  border: 1px solid #83baff55;
  border-radius: 27px;
  background: linear-gradient(125deg, #3979c5 0%, #20549a 48%, #102d5b 100%);
  box-shadow: 0 25px 70px #00000035;
}

.main-weather-content {
  position: relative;
  z-index: 2;
}

.weather-orb {
  position: absolute;
  z-index: -1;
  border-radius: 50%;
  pointer-events: none;
}

.orb-one {
  width: 300px;
  height: 300px;
  top: -150px;
  right: -40px;
  background: #9fcaff15;
}

.orb-two {
  width: 240px;
  height: 240px;
  bottom: -170px;
  left: 32%;
  background: #a8d4ff12;
}

.current-label {
  color: #d3e6ff;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 2px;
}

.current-info h2 {
  margin: 12px 0 3px;
  font-family: 'Manrope', sans-serif;
  font-size: 23px;
  font-weight: 700;
}

.current-date {
  margin: 0;
  color: #d1e1f8;
  font-size: 12px;
}

.temperature-row {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-top: 15px;
}

.main-icon {
  font-size: clamp(55px, 8vw, 86px);
  filter: drop-shadow(0 8px 12px #0a214b33);
}

.temperature {
  font-family: 'Manrope', sans-serif;
  font-size: clamp(68px, 10vw, 100px);
  font-weight: 700;
  line-height: 1;
  letter-spacing: -7px;
}

.temperature span {
  font-weight: 500;
}

.condition {
  margin: 6px 0 16px;
  color: #e0ecff;
  font-size: 14px;
}

.high-low {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}

.high-low span {
  padding: 9px 13px;
  border: 1px solid #d2e6ff30;
  border-radius: 20px;
  background: #061c3c25;
  color: #edf5ff;
  font-size: 12px;
}

.weather-details {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
  margin-top: 32px;
}

.detail-card {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 9px;
  min-width: 0;
  padding: 18px;
  border: 1px solid #d7e9ff2a;
  border-radius: 17px;
  background: #071e3e3b;
  backdrop-filter: blur(10px);
}

.detail-icon {
  font-size: 20px;
}

.detail-label {
  color: #d4e4fa;
  font-size: 11px;
}

.detail-card strong {
  font-family: 'Manrope', sans-serif;
  font-size: 23px;
}

.detail-card small {
  font-size: 10px;
  font-weight: 500;
}

.detail-note {
  color: #b8d1f0;
  font-size: 10px;
}

.weather-decoration {
  position: absolute;
  z-index: 1;
  border: 1px solid #ffffff0b;
  border-radius: 50%;
  pointer-events: none;
}

.decoration-one {
  width: 260px;
  height: 260px;
  top: -100px;
  right: 10%;
}

.decoration-two {
  width: 360px;
  height: 360px;
  top: -150px;
  right: 5%;
}

/* Forecast sections */

.section-block {
  margin-top: 43px;
}

.section-heading {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 18px;
}

.section-heading h2 {
  margin: 0;
  font-family: 'Manrope', sans-serif;
  font-size: 23px;
  font-weight: 700;
  letter-spacing: -0.7px;
}

.section-caption {
  padding-bottom: 4px;
  color: #879ab9;
  font-size: 11px;
}

/* Hourly */

.hourly-list {
  display: grid;
  grid-template-columns: repeat(6, minmax(0, 1fr));
  gap: 12px;
}

.hour-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 17px;
  min-width: 0;
  padding: 19px 8px;
  border: 1px solid #223653;
  border-radius: 18px;
  background: #101c30;
  color: #f5f7ff;
  cursor: pointer;
  transition: background 0.2s, transform 0.2s, border-color 0.2s;
}

.hour-card:hover {
  transform: translateY(-3px);
  border-color: #659cde;
}

.hour-card.active {
  border-color: #7cb9ff;
  background: linear-gradient(180deg, #377dc8, #22518e);
  box-shadow: 0 10px 28px #1a69bd25;
}

.hour-time {
  color: #a8bad3;
  font-size: 11px;
}

.hour-card.active .hour-time {
  color: white;
}

.hour-icon {
  font-size: 27px;
}

.hour-card strong {
  font-family: 'Manrope', sans-serif;
  font-size: 19px;
}

/* Weekly forecast */

.weekly-card {
  padding: 8px 22px;
  border: 1px solid #20334e;
  border-radius: 22px;
  background: #0d192b;
}

.day-row {
  display: grid;
  grid-template-columns: 1.1fr 45px 1.5fr 1.2fr;
  align-items: center;
  gap: 14px;
  min-height: 66px;
  border-bottom: 1px solid #203049;
}

.day-row:last-child {
  border-bottom: none;
}

.day-name {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.day-name strong {
  font-size: 12px;
  font-weight: 600;
}

.day-name span {
  color: #83baff;
  font-size: 10px;
}

.day-icon {
  text-align: center;
  font-size: 24px;
}

.day-condition {
  color: #94a7c2;
  font-size: 11px;
}

.temperature-range {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 10px;
}

.temperature-range strong,
.low-temp {
  min-width: 26px;
  font-size: 12px;
  text-align: right;
}

.low-temp {
  color: #7f91ad;
}

.range-track {
  position: relative;
  overflow: hidden;
  width: 65px;
  height: 5px;
  border-radius: 10px;
  background: #25344b;
}

.range-fill {
  position: absolute;
  top: 0;
  height: 100%;
  border-radius: 10px;
  background: linear-gradient(90deg, #72aaff, #ffc979);
}

.today-row .day-name strong {
  color: #91c4ff;
}

/* Footer */

.footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 15px;
  margin-top: 40px;
  padding: 20px 0 5px;
  border-top: 1px solid #203049;
  color: #7386a3;
  font-size: 11px;
}

.footer-brand {
  color: #c3d8f6;
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 800;
}

.demo-label {
  padding: 5px 8px;
  border: 1px solid #30415a;
  border-radius: 6px;
  color: #9fb1cc;
  font-size: 9px;
  letter-spacing: 1px;
}

/* Tablet */

@media (max-width: 768px) {
  .weather-app {
    padding: 22px 18px;
  }

  .topbar {
    margin-bottom: 40px;
  }

  .main-weather {
    padding: 25px;
    border-radius: 23px;
  }

  .weather-details {
    gap: 8px;
  }

  .detail-card {
    padding: 13px;
  }

  .detail-card strong {
    font-size: 20px;
  }

  .hourly-list {
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }

  .hour-card {
    gap: 12px;
  }

  .day-row {
    grid-template-columns: 1fr 35px 1fr;
    gap: 10px;
    padding: 10px 0;
  }

  .day-condition {
    display: none;
  }

  .temperature-range {
    gap: 7px;
  }
}

/* Mobile */

@media (max-width: 480px) {
  .weather-app {
    padding: 18px 14px;
  }

  .topbar {
    gap: 10px;
    margin-bottom: 34px;
  }

  .location-pill {
    gap: 6px;
    padding: 4px 10px;
    font-size: 10px;
  }

  .city-input {
    width: 90px;
  }

  .profile-button {
    flex-shrink: 0;
    width: 36px;
    height: 36px;
  }

  .page-heading {
    align-items: flex-start;
    flex-direction: column;
    gap: 12px;
    margin-bottom: 20px;
  }

  .page-heading h1 {
    font-size: 34px;
  }

  .live-label {
    font-size: 11px;
  }

  .main-weather {
    padding: 22px 17px;
    border-radius: 21px;
  }

  .current-info h2 {
    font-size: 20px;
  }

  .temperature-row {
    gap: 12px;
    margin-top: 18px;
  }

  .main-icon {
    font-size: 57px;
  }

  .temperature {
    font-size: 74px;
  }

  .high-low {
    gap: 7px;
  }

  .high-low span {
    padding: 8px 10px;
    font-size: 10px;
  }

  .weather-details {
    grid-template-columns: 1fr;
    gap: 9px;
    margin-top: 25px;
  }

  .detail-card {
    display: grid;
    grid-template-columns: 30px 1fr auto;
    align-items: center;
    gap: 5px 9px;
    padding: 12px;
    border-radius: 13px;
  }

  .detail-icon {
    grid-row: span 2;
  }

  .detail-label {
    font-size: 11px;
  }

  .detail-card strong {
    grid-column: 3;
    grid-row: 1 / span 2;
    font-size: 19px;
  }

  .detail-note {
    font-size: 9px;
  }

  .section-block {
    margin-top: 33px;
  }

  .section-heading h2 {
    font-size: 20px;
  }

  .section-caption {
    font-size: 10px;
  }

  .hourly-list {
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 9px;
  }

  .hour-card {
    padding: 15px 5px;
    gap: 12px;
    border-radius: 15px;
  }

  .hour-icon {
    font-size: 25px;
  }

  .hour-card strong {
    font-size: 17px;
  }

  .weekly-card {
    padding: 5px 12px;
    border-radius: 17px;
  }

  .day-row {
    grid-template-columns: minmax(70px, 1fr) 30px minmax(100px, 1fr);
    gap: 5px;
    min-height: 60px;
  }

  .day-name strong {
    font-size: 11px;
  }

  .day-icon {
    font-size: 21px;
  }

  .temperature-range {
    gap: 5px;
  }

  .temperature-range strong,
  .low-temp {
    min-width: 22px;
    font-size: 11px;
  }

  .range-track {
    width: 35px;
  }

  .footer {
    flex-wrap: wrap;
    font-size: 10px;
  }
}
</style>

