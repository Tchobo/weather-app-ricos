
<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <div class="relative mb-8 pt-4">
        <input type="text" placeholder="Search for a city" v-model="searchQuery"
          class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
          @input="getSerchResults" />
        <ul v-if="mapboxSearchResults"
          class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]">
          <p class="text-danger" v-if="searchError">Sorry, something went wrong, please try again</p>
          <p class="text-danger" v-if="!searchError && mapboxSearchResults.length === 0">No results match your query, try a
            different term.</p>
          <template v-else>
            <li v-for="searchResult in mapboxSearchResults" :key="searchResult.id" class="p-y cursor-pointer"
              @click="previewCity(searchResult)">
              {{ searchResult.place_name }}
            </li>
          </template>
        </ul>
      </div>
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
import { ref } from "vue"
import axios from 'axios'
import { useRouter } from 'vue-router'
import CityList from "@/components/CityList.vue"
import CityCardSkeleton from "@/components/CityCardSkeleton.vue"
const router = useRouter()

const mapboxapi = "pk.eyJ1Ijoicmljb3MiLCJhIjoiY2t3amxkMDVwMWhkODJvbnZvcnZhZGx2YSJ9.GTa5GQX-3EAPsjSb3i-C2w"
const searchQuery = ref("");
const queryTimeout = ref(null)
const mapboxSearchResults = ref(null)
const searchError = ref(null)




const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(",")
  console.log(city, state);

  router.push(
    {
      name: 'cityView',
      params: {
        state: state.replace(" ", ""), city: city
      },
      query: {
        lat: searchResult.geometry.coordinates[1],
        lng: searchResult.geometry.coordinates[0],
        preview: true
      },
    }
  );
};

const getSerchResults = () => {
  clearInterval(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(`https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxapi}`);
        mapboxSearchResults.value = result.data.features;
        console.log(mapboxSearchResults.value);
        return;
      }
      catch {
        searchError.value = true
      }
    }
    mapboxSearchResults.value = null;

  }, 300)
}
</script>

