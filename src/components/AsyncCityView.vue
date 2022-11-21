<template>
  <div class="row justify-center bg-indigo-3">
    <q-card class="column justify-center q-my-lg bg-grey-4" style="width: 70%">
      <q-card-section class="column items-center">
        <h2 class="column q-mb-sm q-mt-lg" style="text-align: center">{{weatherDataNow.name}}</h2>
        <h6 class="column q-my-none">
            {{
          new Date(weatherDataNow.currentTime).toLocaleDateString(
            "en-GB",
            {
              weekday: "short",
              day: "2-digit",
              month: "short",
            }
          )
        }}
        {{
          new Date(weatherDataNow.currentTime).toLocaleTimeString(
            "en-GB",
            {
              timeStyle: "short",
            }
          )
        }}
        </h6>
        <h2 class="column text-weight-bold">
            {{Math.round(weatherDataNow.main.temp)}}&deg;
        </h2>
        <div class="column items-center">
            <p class="text-subtitle1 q-mb-none">Feels like: {{Math.round(weatherDataNow.main.feels_like)}}&deg;</p>
            <p class="text-subtitle1 q-mb-none">Description: {{weatherDataNow.weather[0].description}}</p>
            <img
            :src="`http://openweathermap.org/img/wn/${weatherDataNow.weather[0].icon}@2x.png`"
            alt="weather-icon"
            />
        </div>
      </q-card-section>

      <q-separator/>

      <q-card-section class="column items-center q-my-none q-pb-md">
        <h4 class="q-mt-xl q-mb-xl" style="text-align: center">Hourly Weather</h4>    
        <div>
          <q-virtual-scroll :items="weatherDataForecast" virtual-scroll-horizontal style="width: 50vw">
          <template v-slot="{ item, index }">
            <q-item :key="index" class="column items-center" style="width: 70px">
              <p class="text-subtitle1 q-mb-none">
                  {{item.dt_txt.slice(11,16)}}
              </p>
              <img
              :src="`http://openweathermap.org/img/wn/${item.weather[0].icon}@2x.png`"
              alt="hour-weather-icon"
              style="width: 50px"
              />
              <p class="text-subtitle1 text-weight-bold q-ma-none">{{Math.round(item.main.temp)}}&deg;</p>
            </q-item>
          </template>
        </q-virtual-scroll>
        </div>
        
      </q-card-section>
    </q-card>
  </div>
</template>

<script lang="ts" setup>
    import axios from 'axios';
    import { useRoute } from 'vue-router';
    // import { computed, watch } from 'vue'
    
    const route = useRoute();

//     watch(route, (query) => {
//   console.log('Route: ' + query);
// });

    const getWeatherNow = async () => {
        try {
            const weather = await axios.get(`https://pro.openweathermap.org/data/2.5/weather?lat=${route.query.lat}&lon=${route.query.lon}&units=metric&appid=1c849bf90b29b235b780628530433380`)
            
            // get local date & time
            const localOffset = new Date().getTimezoneOffset() * 60000;
            const utc = weather.data.dt * 1000 + localOffset;
            weather.data.currentTime = utc + 1000 * weather.data.timezone;
            return weather.data;
        } catch (err) {
            console.log(err)
        }
    }

    const getWeatherFiveDays = async () => {
        try {
            const weather = await axios.get(`https://pro.openweathermap.org/data/2.5/forecast?lat=${route.query.lat}&lon=${route.query.lon}&units=metric&appid=1c849bf90b29b235b780628530433380`)
            return weather.data.list.slice(0,9);
        } catch (err) {
            console.log(err)
        }
    }

    const weatherDataNow = await getWeatherNow();
    const weatherDataForecast = await getWeatherFiveDays();
</script>