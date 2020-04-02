<template>
  <div id="app">
    <GooglePlacesLocationService
            v-slot="{ suggestions }"
            ref="geoLocate"
            :search="searchInput"
            :country="['ca']"
            :suggestion="selectedSuggestion || null"
            @geocoded="address = { ...$event.normalizedAddress }"
            @error="searchInput = ''"
    >
      <AutoComplete
        :suggestions="suggestions"
        @onSearchInput="getSearchValue"
      />
    </GooglePlacesLocationService>
  </div>
</template>

<script>
import AutoComplete from './components/AutoComplete.vue'
import { loadGmaps, GooglePlacesLocationService } from './components/places/index';

export default {
  name: 'App',
  components: {
    AutoComplete, GooglePlacesLocationService
  },
  data() {
    return {
      searchInput: ''
      };
  },
  mounted() {
      // load google maps API with KEY
      loadGmaps('AIzaSyAJtjrYTb50aVWv1eQrYGFSGAy3-mmEYoE');
  },
  methods: {
    getSearchValue(value) {
      console.log(value);
      this.searchInput = value;
    }
  }
    
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
