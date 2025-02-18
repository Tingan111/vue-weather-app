<script setup>
import{ref}from 'vue'
const city=ref('')
const weather=ref(null)
const fetchWeather=async()=>{
  if(!city.value) return

  const apiKey=import.meta.env.VITE_WEATHER_API_KEY
  const url=`https://opendata.cwa.gov.tw/api/v1/rest/datastore/F-C0032-001?Authorization=${apiKey}&format=JSON`

  
  try {
    const res = await fetch(url)
    const data = await res.json()
  
    if (!data.records || !data.records.location) {
      throw new Error('查詢失敗')
    }

    const location = data.records.location.find(loc => loc.locationName === city.value)

    if (!location) {
      throw new Error('找不到該城市的天氣資訊')
    }

    weather.value = {
      temp: location.weatherElement.find(e => e.elementName === 'MinT').time[0].parameter.parameterName + '°C ~ ' +
            location.weatherElement.find(e => e.elementName === 'MaxT').time[0].parameter.parameterName + '°C',
      description: location.weatherElement.find(e => e.elementName === 'Wx').time[0].parameter.parameterName,
      rain: location.weatherElement.find(e => e.elementName === 'PoP').time[0].parameter.parameterName + '%'
    }
  } catch (error) {
    weather.value = { error: error.message }
  }
}
</script>

<template>
  <div class="container">
    <h1>天氣查詢</h1>
    <input v-model="city" placeholder="請輸入選項城市名稱，例如：臺北市" />
    <button @click="fetchWeather">查詢</button>
    <div v-if="weather !=null">
      <p v-if="weather.error" class="error">{{ weather.error }}</p>
      <div v-else>
        <h2>{{ city }}天氣</h2>
        <p>天氣狀況：{{ weather.description }}</p>
        <p>降雨機率：{{ weather.rain}}</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  text-align: center;
  max-width: 400px;
  margin: auto;
}
.error {
  color: red;
}
</style>
