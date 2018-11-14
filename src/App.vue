<template>
  <div id="app">
    <div class="container">
     
      <div class="mystery-box" ref="mystery" @click="refer" :class="{'hover': name !== ''}">
        <div v-if="name !== ''" class="name">
          {{name}}
        </div>

        <div v-if="rating !== -1" class="rating-box">
            <span class="rating">
            {{rating}}
           </span>
           <img class="star" src="./assets/star.svg">
        </div>
      </div>

      <div class="button" @click="roulette()">
      <div v-if="loading" class="loader">Loading...</div>
        <span v-else>
          Food Roulette!
        </span>
      </div>

      <div v-if="error !== ''" class="error">
        {{error}}
      </div>
<!--
      <div class="inline-buttons">
        <select>
          <option value="volvo">Volvo</option>
          <option value="saab">Saab</option>
          <option value="mercedes">Mercedes</option>
          <option value="audi">Audi</option>
        </select>

        <select>
          <option value="volvo">Volvo</option>
          <option value="saab">Saab</option>
          <option value="mercedes">Mercedes</option>
          <option value="audi">Audi</option>
        </select>
      </div>-->
    </div>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import axios from 'axios';

export default {
  name: 'app',
  components: {
    HelloWorld
  },
  data: function () {
    return {
      lat: null,
      lng: null,
      error: "",
      loading: false,
      name: "",
      link: "",
      rating: -1
    }
  },
  created() {
    this.getGeoLocation();
  },
  mounted() {
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
    getGeoLocation() {
      let self = this;
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(showPosition);
        function showPosition(position) {
          self.lat = position.coords.latitude;
          self.lng = position.coords.longitude;
        }
      }
    },
    getLocaleBusinesses() {
      let self = this;
      axios.get('https://cors-anywhere.herokuapp.com/https://api.yelp.com/v3/businesses/search?latitude=' + this.lat + '&longitude=' + this.lng)
      .then(function (response) {
        // handle success
        self.error = "";
        self.filterBusinesses(response.data);
        console.log(response.data);
      })
      .catch(function (error) {
        // handle error
        self.error = "error";
      });
    },
    filterBusinesses(data) {
      let ran = Math.floor(Math.random() * Math.floor(data.total - 1));
      let b = data.businesses[ran];
      this.name = b["name"];
      this.link = b["url"];
      this.rating = b["rating"].toFixed(1);
      this.$refs.mystery.style.backgroundImage = 'url(' + b["image_url"] + ')';
      this.loading = false;
    }
  }
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Noto+Sans:400,700');

body {
  background: linear-gradient(-45deg, #EE7752, #E73C7E, #23A6D5, #23D5AB);
	background-size: 400% 400%;
	-webkit-animation: Gradient 15s ease infinite;
	-moz-animation: Gradient 15s ease infinite;
	animation: Gradient 15s ease infinite;
}

@-webkit-keyframes Gradient {
	0% {
		background-position: 0% 50%
	}
	50% {
		background-position: 100% 50%
	}
	100% {
		background-position: 0% 50%
	}
}

@-moz-keyframes Gradient {
	0% {
		background-position: 0% 50%
	}
	50% {
		background-position: 100% 50%
	}
	100% {
		background-position: 0% 50%
	}
}

@keyframes Gradient {
	0% {
		background-position: 0% 50%
	}
	50% {
		background-position: 100% 50%
	}
	100% {
		background-position: 0% 50%
	}
}

.container {
  font-family: 'Noto Sans', sans-serif;
  margin-top: 80px;
  text-align: center;
  
  .star {
    width: 70px;
    text-shadow: 0px 0px 50px black;
    display: inline;
    vertical-align: middle;
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
    height: 400px;
    width: 400px;
    margin: 0 auto 0 auto;
    box-shadow: 0px 0px 20px black;
    background: #f9f9f9;
    background-size: cover;
    display: grid;
    .name {
      align-self: start;
      justify-self: center;
      font-size: 40px;
      text-shadow: 1px 1px 10px black;
      color: white;
      width: 400px;
      grid-row-start: 1;
      grid-row-end: 3;
      background-color: #0000009c;
      padding: 10px 0 10px 0;
    }
  }

  .button {
    width: 320px;
    height: 22px;
    margin: 50px auto 10px auto;
    padding: 20px 40px 20px 40px;
    font-weight: 700;
    box-shadow: 0px 0px 20px black;
    color: white;
    background-color: #00000055;
    cursor: pointer;
  }
  .error {

  }
  .hover {
    cursor: pointer;
  }
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

</style>
