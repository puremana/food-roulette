<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'app',
  components: {
    HelloWorld
  },
  data: function () {
    return {
      lat: null,
      lng: null
    }
  },
  created() {
    this.getGeoLocation();
  },
  methods: {
    getGeoLocation() {
      let self = this;
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(showPosition);
        function showPosition(position) {
          self.lat = position.coords.latitude;
          self.lng = position.coords.longitude;
        }
      }
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
