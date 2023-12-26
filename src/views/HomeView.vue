<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        v-if="apiResult"
        class="absoulte bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      >
        <p v-if="serachError">{{ serachError }}</p>
        <p v-if="!serachError && apiResult.length === 0">
          No Match Results. Try Different location
        </p>
        <template v-else>
          <li
            v-for="searchResult in apiResult"
            :key="searchResult.id"
            @click="previewCity(searchResult)"
            class="py-2 cursor-pointer"
          >
            {{ searchResult.display_name }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityList from "../components/CityList.vue";
import CityCardSkeleton from "../components/CityCardSkeleton.vue";

const router = useRouter();
const previewCity = (searchResult) => {
  console.log(searchResult);
  const [city, state] = searchResult.display_name.split(",");
  const lat = searchResult.lat;
  const lon = searchResult.lon;
  const cities = city.replaceAll(" ", "");
  const states = state.replaceAll(" ", "");

  router.push({
    name: "CityView",
    params: {
      state: states,
      city: cities,
    },
    query: {
      lat: lat,
      lng: lon,
      preview: true,
    },
  });
};

const searchQuery = ref("");
const queryTimeout = ref(null);
const apiResult = ref(null);
const serachError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://geocode.maps.co/search?q=${searchQuery.value}&api_key=658acd4e36d1f152143277kyb628e7f`
        );
        apiResult.value = result.data;
      } catch {
        searchError.value = true;
      }
    } else {
      apiResult.value = null;
    }
  }, 200);
};
</script>
