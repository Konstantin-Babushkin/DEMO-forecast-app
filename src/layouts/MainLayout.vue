<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar class="row justify-between">
        <RouterLink :to="'/'" style="text-decoration: none; color: inherit;">
          <q-toolbar-title>
          DEMO Weather App
          </q-toolbar-title>
        </RouterLink>
        <q-input dark dense standout v-model="searchValue" input-class="text-right" class="q-ml-md" @keyup.enter="handleClick">
          <template v-slot:append>
            <q-icon v-if="searchValue === ''" name="search" />
            <q-icon v-else name="clear" class="cursor-pointer" @click="searchValue = ''" />
          </template>
        </q-input>
      </q-toolbar>
      <div class="row justify-end q-mr-lg q-mb-sm">
        <q-list bordered separator v-if="geoResponse" style="width: 200px">
          <q-item clickable v-ripple @click="cityChoice(item)" ref="cityOption" v-for="(item, index) in geoResponse" :key="index">
            {{item.name}}, {{item.country}}
          </q-item>
        </q-list>
        </div>
    </q-header>
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script setup lang="ts">
  import { ref, reactive, watchEffect, onUnmounted } from 'vue';
  import axios from 'axios';
  import {useRouter} from 'vue-router'
  import City from '../components/CityInterface'

  // vars for searching a city name using input
  const router = useRouter();
  const searchValue = ref<string>('');
  const geoResponse = ref<null | City[]>(null)

  // get coordinates by city's name
  const handleClick = async () => {
    if(searchValue.value !== ''){
      const result = await axios.get(
    `https://pro.openweathermap.org/geo/1.0/direct?q=${searchValue.value}&limit=5&appid=1c849bf90b29b235b780628530433380`);
      geoResponse.value = await result.data;
      return;
    } else {
      geoResponse.value = null;
      alert('wrong input')
    }
  }

  // push coordinates and router params to the next view
  const cityChoice = (city : City) => {
    const name : string = city.name;
    const country : string = city.country;
    const lat: number = city.lat;
    const lon: number = city.lon;
    geoResponse.value = null;

    router.push({
      name: 'cityView',
      params: { country: country, city: name },
      query: { lat: lat, lon: lon}
    })    
  }

  // current location
  const latLon = reactive({
    lat: 0,
    lon: 0,
  })
  const coordFound = ref(false)

  // assign current geolocation to latLon object
  const showPosition = (position : GeolocationPosition) => {
    latLon.lat = position.coords.latitude;
    latLon.lon = position.coords.longitude;
    coordFound.value = true;
  };

  // start looking for current geolocation
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(showPosition)
  } else {
      alert(
        'Sorry, no position available. Please, give access to your location'
    );
  }

  // push latLon values (coordinates) to the next view
  const stopWatching = watchEffect(()=>{
    if (coordFound.value) {
      console.log('in render')
      router.push({
        name: 'cityView',
        params: { country: 'currentCounty', city: 'currentCity' },
        query: { lat: latLon.lat, lon: latLon.lon}
      })
      coordFound.value = false;
    }
  })

  onUnmounted(()=>{stopWatching()})
  
</script>
