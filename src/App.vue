<template>
  <div id="app">
    <AutoComplete
    label="Search Countries"
    :items="isExistingUsers"
  />
  </div>
</template>

<script>
import AutoComplete from './components/AutoComplete.vue'

export default {
  name: 'App',
  components: {
    AutoComplete,
  },
  data() {
    let isExistingUsers = [
    { label: 'Canada',},
    { label: 'Cambodia',},
    { label: 'Cameroon',},
    { label: 'Corona'}
    ];
    return {isExistingUsers};
  },
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
