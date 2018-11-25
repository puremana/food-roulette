<template>
  <div id="app">
    <div class="github-container">
      <iframe class="github-button" src="https://ghbtns.com/github-btn.html?user=puremana&repo=food-roulette&type=star&count=true&size=large" frameborder="0" scrolling="0" width="160px" height="30px"></iframe>
    </div>
    <h1>Random Food Picker!</h1>
    <div class="left-container">
      <h2>Options</h2>
      <h3>Price</h3>
      <select v-model="price">
        <option value="0">No Price Filter</option>
        <option value="1">$</option>
        <option value="2">$$</option>
        <option value="3">$$$</option>
        <option value="4">$$$$</option>
      </select>

      <h3>Radius</h3>
      <select v-model="radius">
        <option value="1000">1km</option>
        <option value="2500">2.5km</option>
        <option value="5000">5km</option>
        <option value="10000">10km</option>
        <option value="25000">25km</option>
      </select>

      <h3>Delivery Options</h3>
      <div class="checkbox-container">
        <div class="checkbox">
          <input class="checkbox-input" type="checkbox" name="vehicle1" value="Bike">Delivery
        </div>
        <div class="checkbox">
          <input class="checkbox-input" type="checkbox" name="vehicle1" value="Bike">Pickup
        </div>
      </div>
      
      <h3>Location Address <span class="required">*</span></h3>
      <input type="text" v-model="location" :disabled="geolocation" placeholder="Get food around this address!">
      <div class="switch-container">
        <label class="switch">
          <input ref="geoInput" @click="geolocationCheck" type="checkbox" v-model="geolocation">
          <span class="slider round"></span>
        </label>
        <span>Geolocation</span>
      </div>
      

      <div class="button" @click="roulette()" :class="{'button-active': valid}">
        <div v-if="loading" class="loader">Loading...</div>
        <span v-else>
          Food Roulette!
        </span>
      </div>
    </div>

    <div v-show="name" class="container mystery-box" @click="refer" :class="{'hover': name !== ''}">
      <div class="mystery-box" ref="mystery" @click="refer" :class="{'hover': name !== ''}">
        <div v-if="name !== ''" class="name">
          {{name}}
        </div>

        <div v-if="noImage" class="no-image">
          No Image Found
        </div>

        <div v-if="rating !== -1" class="rating-box">
            <span class="rating">
            {{rating}}
           </span>
           <img class="star" src="./assets/star.svg">
        </div>
      </div>
    </div>

    <div v-if="error !== ''" class="error">
      {{error}}
    </div>

  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'app',
  data: function () {
    return {
      lat: null,
      lng: null,
      error: "",
      loading: false,
      name: "",
      link: "",
      rating: -1,
      price: 0,
      radius: 2500,
      location: "",
      geolocation: false,
      noImage: false
    }
  },
  computed: {
    valid() {
      if (this.location !== "" || (this.lat !== null && this.geolocation)) {
         return true;
      }
      return false;
    }
  },
  created() {
    this.getGeoLocation();
  },
  mounted() {
    if (this.lat !== null) {
      this.geolocation = true;
    }
  },
  methods: {
    refer() {
      if (this.link !== "") {
        window.open(this.link);
      }
    },
    roulette() {
      if (this.loading) {
        return;
      }

      if ((this.lat !== null) && (this.lng !== null)) {
        this.loading = true;
        this.getLocaleBusinesses();
      }
      else {
        this.error = "Please enable geolocation to use the roulette";
      }
      
    },
    geolocationCheck() {
      if (this.geolocation === true) {
        this.geolocation = false;
        return;
      }
      if (this.lat !== null) {
        this.geolocation = true;
      }
      else {
        this.getGeoLocation();
      }
    },
    getGeoLocation() {
      let self = this;
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(showPosition);
        function showPosition(position) {
          self.lat = position.coords.latitude;
          self.lng = position.coords.longitude;
          self.geolocation = true;
        }
      }
    },
    getLocaleBusinesses() {
      let self = this;
      axios.get('https://us-central1-food-roulette-3dd83.cloudfunctions.net/yelpBusinessSearch?lat=' + this.lat + '&lng=' + this.lng + '&radius=' + this.radius + '&price=' + this.price + '&location=' + this.location + '&geo=' + this.geolocation)
      .then(function (response) {
        // handle success
        self.error = "";
        self.filterBusinesses(response.data);
      })
      .catch(function (error) {
        // handle error
        // self.error = "error";
        self.error = error;
      });
    },
    filterBusinesses(data) {
      let ran = Math.floor(Math.random() * Math.floor(data.length - 1));
      console.log(data);
      console.log(data[ran]);
      
      let b = data[ran];
      console.log("what " + b["image_url"]);
      this.name = b["name"];
      this.link = b["url"];
      this.rating = b["rating"].toFixed(1);
      this.noImage = false;
      if (b["image_url"] === "") {
        this.noImage = true;
      }
      this.$refs.mystery.style.backgroundImage = "url('" + b["image_url"] + "')";
      console.log(this.$refs.mystery);
      this.loading = false;
    }
  }
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Noto+Sans:400,700');

body {
  font-family: 'Noto Sans', sans-serif;
  text-align: center;
  h1 {
    margin-bottom: 30px;
  }
  .github-container {
    position: relative;
    .github-button {
      position: absolute;
      right: 0;
      margin-bottom: -20px;
    }
  }
}

.left-container {
  box-shadow: 0px 0px 20px black;
  display: inline-block;
  vertical-align: top;
  max-width: 475px;
  width: 90vw;
  min-width: 300px;
  
  margin: 0 1% 0 1%;
  margin-bottom: 20px;
  text-align: center;
  min-height: 475px;
  position: relative;

  h2 {
    margin: 20px 0 0 0;
  }
  select, input {
    padding: 10px;
    margin-bottom: 5px;
    font-family: 'Noto Sans', sans-serif;
  }
  select {
    width: 90%;
  }
  input {
    width: calc(90% - 20px);
  }
  .checkbox-input {
    width: 40px;
  }
  .checkbox {
    width: 120px;
    display: inline-block;
  }
  .checkbox-container {
    text-align: left;
    margin-left: 5%;
    margin-bottom: 5px;
  }
  h3 {
    text-align: left;
    margin: 0 0 5px 5%;
    font-weight: 400;
  }
  .required {
    color: #df0909;
  }
  .button {
    width: 90%;
    height: 22px;
    position: absolute;
    bottom: -10px;
    margin: 0 auto 0 auto;
    left: 50%;
    transform: translate(-50%, -50%);
    padding: 20px 0 20px 0;
    font-weight: 700;
    color: white;
    background-color: #aaaaaa;
  }
  .button-active {
    cursor: pointer;
    background-color: #2196F3;
  }
}

.container {
  margin: 0 1% 0 1%;
  vertical-align: top;
  display: inline-block;
  .star {
    width: 70px;
    text-shadow: 0px 0px 50px black;
    display: inline;
    vertical-align: middle;
  }

  .no-image {
    font-size: 35px;
    color: #878484;
  }

  .rating {
    font-size: 40px;
    color: white;
    display: inline;
    align-self: center;
    line-height: 70px;
    vertical-align: middle;
    text-shadow: 1px 1px 10px black;
  }

  .rating-box {
    bottom: 0px;
    right: 0px;
    grid-template-columns: 1fr 1fr;
    align-self: end;
    justify-self: end;
    align-items: center;
    justify-items: center;
    padding-right: 10px;
    padding-bottom: 10px;
  }

  .mystery-box {
    box-shadow: 0px 0px 20px black;
      max-height: 475px;
      height: 90vw;
    // width: 400px;
    width: 90vw;
    max-width: 475px;
    min-width: 300px;
    min-height: 300px;
    background: #f9f9f9;
    background-size: cover;
    display: grid;
    .name {
      align-self: start;
      justify-self: center;
      font-size: 40px;
      text-shadow: 1px 1px 10px black;
      color: white;
      width: 100%;
      grid-row-start: 1;
      grid-row-end: 3;
      background-color: #0000009c;
      padding: 10px 0 10px 0;
    }
  }
  .error {

  }

}

.hover {
  cursor: pointer;
}


.loader {
  position: absolute; 
  left: 0; 
  right: 0; 
  margin-left: auto; 
  margin-right: auto;
  color: #ffffff;
  font-size: 20px;
  text-indent: -9999em;
  overflow: hidden;
height: 1em;
width: 1em;
  border-radius: 50%;
  position: absolute;
  -webkit-transform: translateZ(0);
  -ms-transform: translateZ(0);
  transform: translateZ(0);
  -webkit-animation: load6 1.7s infinite ease, round 1.7s infinite ease;
  animation: load6 1.7s infinite ease, round 1.7s infinite ease;
}
@-webkit-keyframes load6 {
  0% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em, 0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
  5%,
  95% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em, 0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
  10%,
  59% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.087em -0.825em 0 -0.42em, -0.173em -0.812em 0 -0.44em, -0.256em -0.789em 0 -0.46em, -0.297em -0.775em 0 -0.477em;
  }
  20% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.338em -0.758em 0 -0.42em, -0.555em -0.617em 0 -0.44em, -0.671em -0.488em 0 -0.46em, -0.749em -0.34em 0 -0.477em;
  }
  38% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.377em -0.74em 0 -0.42em, -0.645em -0.522em 0 -0.44em, -0.775em -0.297em 0 -0.46em, -0.82em -0.09em 0 -0.477em;
  }
  100% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em, 0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
}
@keyframes load6 {
  0% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em, 0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
  5%,
  95% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em, 0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
  10%,
  59% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.087em -0.825em 0 -0.42em, -0.173em -0.812em 0 -0.44em, -0.256em -0.789em 0 -0.46em, -0.297em -0.775em 0 -0.477em;
  }
  20% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.338em -0.758em 0 -0.42em, -0.555em -0.617em 0 -0.44em, -0.671em -0.488em 0 -0.46em, -0.749em -0.34em 0 -0.477em;
  }
  38% {
    box-shadow: 0 -0.83em 0 -0.4em, -0.377em -0.74em 0 -0.42em, -0.645em -0.522em 0 -0.44em, -0.775em -0.297em 0 -0.46em, -0.82em -0.09em 0 -0.477em;
  }
  100% {
    box-shadow: 0 -0.83em 0 -0.4em, 0 -0.83em 0 -0.42em, 0 -0.83em 0 -0.44em, 0 -0.83em 0 -0.46em, 0 -0.83em 0 -0.477em;
  }
}
@-webkit-keyframes round {
  0% {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
@keyframes round {
  0% {
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}

.switch-container {
  text-align: left;
    span {
    color: grey;
  }
}
.switch {
  position: relative;
  display: inline-block;
  width: 36px;
  height: 20px;
  margin-left: 5%;
  margin-right: 10px;
}

.switch input { 
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  -webkit-transition: .4s;
  transition: .4s;
}

.slider:before {
  position: absolute;
  content: "";
  height: 14px;
  width: 14px;
  left: 4px;
  bottom: 3px;
  background-color: white;
  -webkit-transition: .4s;
  transition: .4s;
}

input:checked + .slider {
  background-color: #2196F3;
}

input:focus + .slider {
  box-shadow: 0 0 1px #2196F3;
}

input:checked + .slider:before {
  -webkit-transform: translateX(14px);
  -ms-transform: translateX(14px);
  transform: translateX(14px);
}

/* Rounded sliders */
.slider.round {
  border-radius: 20px;
}

.slider.round:before {
  border-radius: 50%;
}

@media only screen and (max-width: 680px) {
  .github-button {
    // position: relative !important;
  }
  h1 {
    padding-top: 40px;
  }
}
</style>
