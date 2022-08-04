<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search / Results -->
    <div class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32">
      <!-- Search Input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input 
          v-model="queryIp"
          class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none" 
          type="text" 
          placeholder="Search for any IP address or leave empty to get your IP info">
          <i class="fas fa-chevron-right cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center"
          @click="getIpInfo"></i>
        </div>
      </div>
      <!-- IP Info -->
      <IPInfo v-if="ipInfo" :ipInfo="ipInfo" />
    </div>

    <div id="map" class="h-full z-10"></div>
  </div>

</template>

<script setup>
import IPInfo from '@/components/IPInfo.vue';
import leaflet from 'leaflet';
import {onMounted, ref} from 'vue'

const mapToken = import.meta.env.VITE_MAP_TOKEN;
const geolocKey = import.meta.env.VITE_GEOLOC_KEY;

let map;
const queryIp = ref('');
const ipInfo = ref(null);

onMounted(() => {
  map = leaflet.map('map').setView([55.8, 38], 9);
  leaflet.tileLayer(`https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${mapToken}`,
          {
            attribution:
              'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
            maxZoom: 18,
            id: "mapbox/streets-v11",
            tileSize: 512,
            zoomOffset: -1,
            accessToken: mapToken,
          }).addTo(map);
})

const getIpInfo = async () => {
  try {
    const res = await fetch(`https://api.ipgeolocation.io/ipgeo?apiKey=${geolocKey}&ip=${queryIp.value}`);
    const data = await res.json();
    ipInfo.value = {
      address: data.ip,
      state: data.state_prov,
      timezone: data.time_zone.offset,
      isp: data.isp,
      lat: data.latitude,
      lng: data.longitude,
    }

    leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(map);
    map .setView([ipInfo.value.lat, ipInfo.value.lng], 13);
  } catch (err) {
    console.log(err.message)
  }
}

</script>
