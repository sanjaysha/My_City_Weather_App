<template>
  <div>
    <div class="mb-3" v-for="city in savedCities" :key="city.id">
      <CityCard :city="city" @click="goToCityView(city)" />
    </div>
    <p v-if="savedCities.length === 0">
      No locations added. To start tracking a location, search in the field
      above.
    </p>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import CityCard from "./CityCard.vue";
import { useRouter } from "vue-router";

const savedCities = ref([]);
const getCities = async () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));
  }

  const requests = [];
  savedCities.value.forEach((city) => {
    requests.push(
      axios.get(
        `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=e06ae03734e1b0e93763527786dd30ee&units=imperial`
      )
    );
  });

  const weatherData = await Promise.all(requests);

  //   flicker delay
  await new Promise((res) => setTimeout(res, 1000));
  
  weatherData.forEach((value, index) => {
    savedCities.value[index].weather = value.data;
  });
};

await getCities();

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: "cityView",
    params: {
      city: city.city,
      state: city.state,
    },
    query: {
      lat: city.coords.lat,
      lng: city.coords.lng,
      id: city.id,
    },
  });
};
</script>
