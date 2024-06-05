<script setup lang="ts">
import { Geolocation } from '@capacitor/geolocation';

const location = ref('Latitude: 0, Longitude: 0');
const latitude = ref();
const longitude = ref();
const oldLatitude = ref();
const oldLongitude = ref();
const distance = ref(0);

async function getLocation () {
  oldLatitude.value = latitude.value;
  oldLongitude.value = longitude.value;

  setInterval(() => {
    Geolocation.getCurrentPosition().then((position) => {
      latitude.value = position.coords.latitude;
      longitude.value = position.coords.longitude;
      location.value = `Latitude: ${latitude.value}, Longitude: ${longitude.value}`;

      if(oldLatitude.value && oldLongitude.value) {
        const dist = calculateDistance(
          { latitude: oldLatitude.value, longitude: oldLongitude.value },
          { latitude: latitude.value, longitude: longitude.value }
        );
        distance.value += dist;
      }

      oldLatitude.value = latitude.value;
      oldLongitude.value = longitude.value;
    });
  }, 5000);
}

function calculateDistance(coord1: { latitude: number; longitude: number; }, coord2: { latitude: number; longitude: number; }) {
  const R = 6371e3; // raio médio da Terra em metros
  const lat1Rad = coord1.latitude * Math.PI/180;
  const lat2Rad = coord2.latitude * Math.PI/180;
  const deltaLat = (coord2.latitude-coord1.latitude) * Math.PI/180;
  const deltaLon = (coord2.longitude-coord1.longitude) * Math.PI/180;

  const a = Math.sin(deltaLat/2) * Math.sin(deltaLat/2) +
            Math.cos(lat1Rad) * Math.cos(lat2Rad) *
            Math.sin(deltaLon/2) * Math.sin(deltaLon/2);
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));

  const distance = R * c;
  return distance;
}

function getPermission() {
  Geolocation.requestPermissions().then((permission) => {
    if(permission.location === 'denied') {
      // eslint-disable-next-line no-alert
      alert('Você precisa permitir o acesso a localização para continuar');
    }
  });
}

onMounted(() => {
  getPermission();
  getLocation();
});
</script>

<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>
          <span class="font-bold text-green-500">NUXT</span> ✨ + <span class="font-bold text-blue-500">IONIC</span> ✨
        </ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <main class="space-y-10">
          <h1 class="text-2xl text-center">Geolocalização</h1>
          <p>{{ location }}</p>
          <p>Distância percorrida: {{ distance }} metros</p>
      </main>
    </ion-content>
  </ion-page>
</template>
