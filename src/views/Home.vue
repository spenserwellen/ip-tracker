<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search / Results -->
    <div
      class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32"
    >
      <!-- Search Input -->
      <div class="w-full max-w-screen-sm">
        <a href="/">
          <h1 class="text-white text-center text-3xl pb-4">
            IP Address Tracker
          </h1>
        </a>
        <div class="flex">
          <input
            v-model="queryIp"
            class="flex-1 py-3 px-2 rounded-tl-md rounded-bl-md focus:outline-none"
            type="text"
            placeholder="Search for any IP address or leave empty to get your ip info"
            @keypress="fetchLocation"
          />
          <i
            @click="getIpInfo"
            class="cursor-pointer
          bg-black
          text-white
          px-4
          rounded-tr-md
          rounded-br-md
          flex
          items-center
          fas fa-chevron-right"
          ></i>
        </div>
      </div>
      <!-- IP Info -->
      <IPInfo v-if="ipInfo" v-bind:ipInfo="ipInfo" />
    </div>

    <!-- Map -->
    <div id="mapid" class="h-full z-10"></div>
  </div>
</template>

<script>
import IPInfo from '../components/IPInfo.vue';
import leaflet from 'leaflet';
import { onMounted, ref } from 'vue';
import axios from 'axios';
export default {
  name: 'Home',
  components: { IPInfo },
  setup() {
    // create map variable
    let mymap;
    // data
    const queryIp = ref('');
    const ipInfo = ref(null);
    // mounted lifecycle hook, creates the map
    onMounted(() => {
      mymap = leaflet.map('mapid').setView([45.5, -122.675], 11);
      leaflet
        .tileLayer(
          `https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${process.env.VUE_APP_MAPBOX_API_ACCESS_TOKEN}`,
          {
            attribution:
              'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
            maxZoom: 18,
            id: 'mapbox/streets-v11',
            tileSize: 512,
            zoomOffset: -1,
            accessToken: process.env.VUE_APP_MAPBOX_API_ACCESS_TOKEN,
          }
        )
        .addTo(mymap);
    });
    // gets ip information from API
    const getIpInfo = async () => {
      try {
        const data = await axios.get(
          `https://geo.ipify.org/api/v1?apiKey=${process.env.VUE_APP_GEO_IPIFY_API_KEY}&ipAddress=${queryIp.value}`
        );
        const result = data.data;
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng,
        };
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
        mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
      } catch (err) {
        alert(err.message);
      }
    };
    // for keypress of enter on the input
    const fetchLocation = async (e) => {
      if (e.key == 'Enter') {
        try {
          const data = await axios.get(
            `https://geo.ipify.org/api/v1?apiKey=${process.env.VUE_APP_GEO_IPIFY_API_KEY}&ipAddress=${queryIp.value}`
          );
          const result = data.data;
          ipInfo.value = {
            address: result.ip,
            state: result.location.region,
            timezone: result.location.timezone,
            isp: result.isp,
            lat: result.location.lat,
            lng: result.location.lng,
          };
          leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(mymap);
          mymap.setView([ipInfo.value.lat, ipInfo.value.lng], 13);
        } catch (err) {
          alert(err.message);
        }
      }
    };
    return { queryIp, ipInfo, getIpInfo, fetchLocation };
  },
};
</script>
