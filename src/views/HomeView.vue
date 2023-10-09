<script setup>
import { ref } from 'vue';
import axios from 'axios';
import { useRouter } from 'vue-router';

const router = useRouter();

const previewCity = (searchResult) => {
  console.log(searchResult);
  const [city, state] = searchResult.place_name.split(', ');
  router.push({
    name: 'cityView',
    params: { state, city },
    query:{
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview:true,
    }
  });
};

const mapBoxAPIKey =
  'pk.eyJ1Ijoib2xla3NhbmRyaGlyeWNoIiwiYSI6ImNsbmVoZzRsaTBkZXAya3F4NDltbW4wM3YifQ.22ZB9gwTWnyp4FgHonr0AQ';
const searchQuery = ref('');

const queryTimeout = ref(null);
const searchResults = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value) {
      const result = await axios.get(
        `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapBoxAPIKey}&types=place`
      );

      searchResults.value = result.data.features;
      console.log(searchResults.value);
      return;
    }

    searchResults.value = null;
  }, 300);
};
</script>

<template>
  <main class="container text-white">
    <div class="p-4 mb-8 relative">
      <input
        type="text"
        @input="getSearchResults"
        v-model="searchQuery"
        placeholder="Search for city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus: outline-none focus:shadow-md"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="searchResults"
      >
        <li
          v-for="searchResult in searchResults"
          :key="searchResult.id"
          class="py-2 cursor-pointer"
          @click="previewCity(searchResult)"
        >
          {{ searchResult.place_name }}
        </li>
      </ul>
    </div>
  </main>
</template>
