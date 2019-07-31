<template>
  <div id="app">
    <a href="https://www.yelp.com/" rel="noopener noreferrer" target="_blank">
      <img src="./assets/yelp-logo.png" class="yelp-logo" alt="Yelp Logo">
    </a>
    <div class="github-container">
      <iframe class="github-button" src="https://ghbtns.com/github-btn.html?user=puremana&repo=food-roulette&type=star&count=true&size=large" frameborder="0" scrolling="0" width="160px" height="30px"></iframe>
    </div>
    <h1>Random Food Picker!</h1>
    <div class="left-container" v-bind:class="{ 'left-contaner-filters': filters }">
      <h2>Options</h2>
      <h3 v-if="filters">Price</h3>
      <select v-if="filters" v-model="price" @change="inputChanged">
        <option value="0">No Price Filter</option>
        <option value="1">$</option>
        <option value="2">$$</option>
        <option value="3">$$$</option>
        <option value="4">$$$$</option>
      </select>

      <h3>Radius</h3>
      <select v-model="radius" @change="inputChanged">
        <option value="1000">1km</option>
        <option value="2500">2.5km</option>
        <option value="5000">5km</option>
        <option value="10000">10km</option>
        <option value="25000">25km</option>
      </select>

      <h3 v-if="filters">Delivery Options</h3>
      <div v-if="filters" class="checkbox-container">
        <div class="checkbox">
          <input class="checkbox-input" type="checkbox" v-model="delivery" @change="inputChanged">Delivery
        </div>
        <div class="checkbox">
          <input class="checkbox-input" type="checkbox" v-model="pickup" @change="inputChanged">Pickup
        </div>
      </div>
      
      <h3>Location Address <span class="required">*</span></h3>
      <input type="text" id="search" ref="search" :disabled="geolocation" placeholder="Get food around this address!" @input="inputSearchInput" @change="inputChanged">
      <div class="switch-container">
        <label class="switch">
          <input ref="geoInput" @click="geolocationCheck()" type="checkbox" v-bind:checked="geolocation" @change="inputChanged">
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

        <div v-if="phoneNumber" class="phone-number">
          {{phoneNumber}}
        </div>

        <div v-if="rating !== -1" class="rating-box">
           <img class="star" :src="ratingPicture">
           <p class="review-text">Based on {{ reviews }} Reviews</p>
        </div>
      </div>
    </div>

    <div v-if="error" class="error">
      {{error}}
    </div>

    <div ref="map" class="map-container"></div>
    <div v-if="name" style="color: white;">
      Distance From You: {{distance}}m
    </div>

  </div>
</template>

<script>
import axios from 'axios';
import places from 'places.js';

export default {
  name: 'app',
  data: function () {
    return {
      filters: false,
      lat: null,
      lng: null,
      slat: null,
      slng: null,
      error: "",
      loading: false,
      name: "",
      link: "",
      rating: -1,
      price: 0,
      radius: 2500,
      geolocation: false,
      noImage: false,
      delivery: false,
      pickup: false,
      phoneNumber: null,
      reviews: 0,
      searchInput: "",
      ssearchInput: null,
      searchValue: "",
      changed: false,
      searches: 0,
      filteredBusinesses: []
    }
  },
  computed: {
    searchValid() {
      if (this.searchInput === this.ssearchInput) {
        return true;
      }
      return false;
    },
    valid() {
      if (this.searchValid || (this.lat !== null && this.geolocation)) {
         return true;
      }
      return false;
    },
    zoom() {
      switch(this.radius) {
        case 1000:
          return 15;
          break;
        case 2500:
          return 14;
          break;
        case 5000:
          return 14;
          break;
        case 10000:
          return 13;
          break;
        default:
          return 12;
      }
    },
    rlat() {
      if (this.geolocation) {
        if (this.lat) {
          return this.lat;
        }
      }
      return this.slat;
    },
    rlng() {
      if (this.geolocation) {
        if (this.lng) {
          return this.lng;
        }
      }
      return this.slng;
    },
    ratingPicture() {
      if (this.rating === -1) {
        return null;
      }
      return require("./assets/ratings/" + this.rating.replace(".", "-") + ".png");
    }
  },
  created() {
    this.getGeoLocation();
  },
  mounted() {
    if (this.lat !== null) {
      this.geolocation = true;
    }

    const fixedOptions = {
      appId: 'plG5I3EZLNQY',
      apiKey: '4e7eaf7c20fb1520e32886afaf0d13b0',
      container: document.querySelector('#search')
    };

    const placesInstance = places(fixedOptions);

    let self = this;
    placesInstance.on('change', function(e) {
      this.searchInput = true;
      self.slat = e.suggestion.latlng.lat;
      self.slng = e.suggestion.latlng.lng;
      self.ssearchInput = e.suggestion.query;
    });

    let clearbutton = document.getElementsByClassName("ap-icon-clear");
    clearbutton[0].onclick = function() {
     self.searchInput = "";
    }

  },
  methods: {
    inputChanged() {
      this.changed = true;
    },
    // Get geolocation when page is created
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
    
    // Methods relating to interactions on the picker aside from Food Roulette
    geolocationCheck(event) {
      if (event) event.preventDefault()
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
    inputSearchInput() {
      this.searchInput = this.$refs.search.value;
    },
    refer() {
      if (typeof ga === 'function') {
        ga('send', 'event', 'Yelp Image', 'Click', true);
      }
      if (this.link !== "") {
        window.open(this.link);
      }
    },

    // Food Roulette Logic
    roulette() {
      if (this.loading || !this.valid) {
        return;
      }

      if (!this.changed && this.searches > 0 && this.filteredBusinesses.length > 0) {
        this.filterBusinesses(this.filteredBusinesses);
        return;
      }

      if (typeof ga === 'function') {
        ga('send', 'event', 'Roulette Button', 'Submit', true);
        ga('send', 'event', 'Geo', 'Value', this.geolocation);
        ga('send', 'event', 'Price', 'Value', this.price);
        ga('send', 'event', 'Radius', 'Value', this.radius);
        ga('send', 'event', 'Delivery/Pickup', 'Value', this.delivery + "," + this.pickup);
      }
      
      this.loading = true;
      this.getLocaleBusinesses();
    },
    getLocaleBusinesses() {
      let self = this;
      axios.get('https://us-central1-food-roulette-3dd83.cloudfunctions.net/yelpBusinessSearch?lat=' + this.rlat + '&lng=' + this.rlng + '&radius=' + this.radius + '&price=' + this.price)
      .then(function (response) {
        // handle success
        // Reset input changed variable for caching
        self.changed = false;
        self.searches++;
        self.error = "";
        if (response.data.length === 0) {
          self.error = "No open businesses were found matching your options, try increasing your radius!";
        } else if (response.data.statusCode === 400) {
          self.error = "Location not found, try specifying a more exact location!";
        }
        else {
          let realData = self.filterData(response.data);
          self.filteredBusinesses = realData;
          if (realData.length === 0) {
            self.error = "No open businesses were found matching your options, try increasing your radius!";
          }
          else {
            self.filterBusinesses(realData);
          }
        }
        self.loading = false;
      })
      .catch(function (error) {
        self.error = "Error: " + error;
      });
    },
    filterData(data) {
      let newData = [];
      for (let d in data) {
        if (data[d]["is_closed"]) {
          continue;
        }
        if (this.delivery) {
          if (!data[d].transactions.includes("delivery")) {
            continue;
          }
        }
        if (this.pickup) {
          if (!data[d].transactions.includes("pickup")) {
            continue;
          }
        }
        newData.push(data[d]);
      }
      return newData;
    },
    filterBusinesses(data) {
      let ran = Math.floor(Math.random() * Math.floor(data.length - 1));
      let b = data[ran];
      console.log(JSON.stringify(b));
      this.name = b["name"];
      this.link = b["url"];
      this.rating = b["rating"].toFixed(1);
      this.reviews = b["review_count"];
      this.noImage = false;
      if (b["image_url"] === "") {
        this.noImage = true;
      }
      this.phoneNumber = b["display_phone"];
      this.$refs.mystery.style.backgroundImage = "url('" + b["image_url"] + "')";
      this.distance = Math.round(b["distance"]);

      this.createMapsLocation(b["coordinates"]["latitude"], b["coordinates"]["longitude"]);
    },
    createMapsLocation(x, y) {
      if (document.getElementById("mapid")) {
        let element = document.getElementById("mapid");
        element.parentNode.removeChild(element);
      }
      let m = document.createElement("div");
      m.setAttribute("id", "mapid");
      this.$refs.map.appendChild(m);

      let mymap = L.map('mapid').setView([this.rlat, this.rlng], this.zoom);
      L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token=pk.eyJ1IjoicHVyZW1hbmEiLCJhIjoiY2pvd2N1eWw5MDlpYjNwbXRocjN3bTJmayJ9.Wi_6gEx-kcS2uq1_OU_Jew', {
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        maxZoom: 30,
        id: 'mapbox.streets',
        accessToken: 'pk.eyJ1IjoicHVyZW1hbmEiLCJhIjoiY2pvd2N1eWw5MDlpYjNwbXRocjN3bTJmayJ9.Wi_6gEx-kcS2uq1_OU_Jew'
      }).addTo(mymap);

      let marker = L.marker([x, y]).addTo(mymap);

      let circle = L.circle([this.rlat, this.rlng], {
          color: '#00d667',
          fillColor: '#00d667',
          fillOpacity: 0.1,
          radius: this.radius
      }).addTo(mymap);

        let popup = L.popup()
      .setLatLng([this.rlat, this.rlng])
      .setContent("You are here!")
      .openOn(mymap);
    }
  }
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Noto+Sans:400,700');

body {
  font-family: 'Noto Sans', sans-serif;
  text-align: center;

  background: linear-gradient(128deg, #00b3ff, #006fff);
  background-size: 400% 400%;

  -webkit-animation: backgroundAnimation 10s ease infinite;
  -moz-animation: backgroundAnimation 10s ease infinite;
  animation: backgroundAnimation 10s ease infinite;

  @-webkit-keyframes backgroundAnimation {
      0%{background-position:0% 4%}
      50%{background-position:100% 50%}
      100%{background-position:0% 4%}
  }
  @-moz-keyframes backgroundAnimation {
      0%{background-position:0% 4%}
      50%{background-position:100% 50%}
      100%{background-position:0% 4%}
  }
  @keyframes backgroundAnimation { 
      0%{background-position:0% 4%}
      50%{background-position:100% 50%}
      100%{background-position:0% 4%}
  }

  .yelp-logo {
    width: 100px;
    display: inline;
  }

  h1 {
    color: white;
    margin-bottom: 30px;
    padding-top: 0;
    margin-top: 0;
  }
  .github-container {
    position: absolute;
    display: inline-block;
    right: 0;
    top: 30px;
    .github-button {
      position: absolute;
      right: 0;
      margin-bottom: -20px;
    }
  }
}

.ap-icon-pin {
  display: none;
}

.ap-icon-clear {
  right: 20px;
  margin-right: 20px;
  margin-bottom: 4px;
}

.ap-input, .ap-dropdown-menu {
  font-family: 'Noto Sans', sans-serif !important;
}

.error {
  box-shadow: 0px 0px 20px black;
  border: 2px solid red;
  background: #ffcaca;
  padding: 20px 10px;
  max-width: 455px;
  margin: 0 auto 20px auto;
}

#mapid {
  box-shadow: 0px 0px 20px black;
  max-width: 980px;
  height: 400px;
  margin: 0 auto 0 auto;
}

.map-container {
  margin: 0 3.5% 10px 3.5%;
}

.left-container {
  background: white;
  box-shadow: 0px 0px 20px black;
  display: inline-block;
  vertical-align: top;
  max-width: 475px;
  width: 90vw;
  min-width: 300px;
  
  margin: 0 1% 0 1%;
  margin-bottom: 20px;
  text-align: center;
  min-height: 360px;
  position: relative;

  h2 {
    margin: 20px 0 0 0;
  }
  select, input {
    padding: 10px;
    margin-bottom: 5px;
    font-family: 'Noto Sans', sans-serif;
    border: 1px solid #CCC;
  }
  select {
    width: 90%;
  }
  input {
    width: calc(95% - 20px);
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
.left-contaner-filters {
  min-height: 480px;
}

.container {
  margin: 0 1% 20px 1%;
  position: relative;
  vertical-align: top;
  display: inline-block;
  .star {
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
    .review-text {
      text-shadow: 1px 1px 10px black;
      color: white;
      text-align: right;
      margin: 5px 5px 0 0;
    }
  }

  .phone-number {
    position: absolute;
    bottom: 0;
    left: 0;
    padding-left: 10px;
    padding-bottom: 10px;
    font-size: 30px;
    text-shadow: 1px 1px 10px black;
    color: white;
  }

  .mystery-box {
    box-shadow: 0px 0px 20px black;
    max-height: 475px;
    height: 90vw;
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

.slider.round {
  border-radius: 20px;
}

.slider.round:before {
  border-radius: 50%;
}

@media only screen and (max-width: 680px) {
  h1 {
    padding-top: 40px;
  }
}
@media only screen and (max-width: 680px) {
  .yelp-logo {
    position: absolute;
    left: 15%;
  }
  h1 {
    padding-top: 70px !important;
  }
}
@media only screen and (max-width: 495px) {
  .container .phone-number {
    font-size: 26px;
  }
}
@media only screen and (max-width: 450px) {
  .container .phone-number {
    font-size: 22px;
  }
  .container .mystery-box .name {
    font-size: 30px;
  }
}
@media only screen and (max-width: 405px) {
  h1 {
    font-size: 1.5em;
  }
  .container .phone-number {
    font-size: 18px;
  }
  .container .mystery-box .name {
    font-size: 26px;
  }
}
</style>
