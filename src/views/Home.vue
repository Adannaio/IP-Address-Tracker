<template>
  <div class="flex flex-col h-screen max-h-screen">
    <div class="flex z-20 justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32">

      <!-- Search Input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4">IP Address Tracker</h1>
        <div class="flex">
          <input type="text" class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none" v-model="queryIp" placeholder="Get your IP info">
          <i @click="getIpInfo" class="cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center fas fa-chevron-right"></i>
        </div>
      </div>
      <IPinfo v-if="ipInfo" :ipInfo="ipInfo"/>
    </div>

    <!-- Map -->
    <div id="map" class="h-full z-10"></div>
  </div>
</template>


<script>

import IPinfo from "../components/IPinfo.vue";
import leaflet from "leaflet";
import {onMounted, ref} from 'vue';
import axios from "axios";

export default {
  name: 'Home',
  components: {IPinfo},
  setup() {
    let mymap;

    const queryIp = ref("");
    const ipInfo = ref(null)
    onMounted(() => {
      mymap = leaflet.map('map').setView([51.505, -0.09], 9);
      leaflet
        .tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoiYWRhbm5haW8iLCJhIjoiY2t3OTM3cDg3MGtpNjJ4cDJ1ZG14b2dsciJ9.0TAdG5a86ZsPQjHZaOvmyw', {
          attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
          maxZoom: 18,
          id: 'mapbox/streets-v11',
          tileSize: 512,
          zoomOffset: -1,
          accessToken: 'pk.eyJ1IjoiYWRhbm5haW8iLCJhIjoiY2t3OTM3cDg3MGtpNjJ4cDJ1ZG14b2dsciJ9.0TAdG5a86ZsPQjHZaOvmyw'
      }).addTo(mymap);
    });
    
    const getIpInfo = async () => {
      try {
        const response = await axios.get(`https://geo.ipify.org/api/v2/country?apiKey=at_oIRSoItIHLq2EYSYDSOPjt4uo478O&ipAddress=${queryIp.value}`);
        const result = response.data;
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone:result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng
        }
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
      } catch (error) {
        alert(error.message);
      }
    }
    
    return { queryIp, ipInfo, getIpInfo }

  },
}
</script>
