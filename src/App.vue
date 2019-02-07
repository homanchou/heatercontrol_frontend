<template>
  <div id="app">
    <h2 v-if="errMsg">There was an error loading heater data</h2>
    <div v-if="loaded">
      <h1>Heater Control
        <toggle-button :value="heaterState.enabled" @change="toggleHeater"/>
      </h1>
      <div id="currentTemp">{{heaterState.temp | wholeNum}}</div>
      <div v-if="heaterState.enabled">
        <a class="adjustTemp" @click="makeColder">ᐯ</a>
        <span id="desiredTemp">{{heaterState.desired_temp | wholeNum}}</span>
        <a class="adjustTemp" @click="makeHotter">ᐱ</a>
      </div>
      {{(heaterState.heater_on) ? "heater is on" : "heater is off"}}
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { ToggleButton } from "vue-js-toggle-button";
import TemperatureSlider from "./components/TemperatureSlider";

let root = "";
//  "http://localhost:5000";
export default {
  name: "app",
  components: { ToggleButton },
  data() {
    return {
      errMsg: null,
      loaded: false,
      heaterState: {}
    };
  },
  filters: {
    wholeNum: function(num) {
      return Math.round(num);
    }
  },
  created() {
    axios
      .get(root + "/status")
      .then(result => {
        this.loaded = true;
        this.heaterState = result.data;
        console.log("result is", result);
      })
      .catch(err => {
        this.errMsg = err;
        console.log("some error occured", err);
      });
  },
  methods: {
    setDesiredTemp: function() {
      axios
        .post(root + "/set_desired_temp/" + this.heaterState.desired_temp)
        .then(result => {
          this.loaded = true;
          this.heaterState = result.data;
          console.log("set desired temp", result);
        });
    },
    makeHotter: function() {
      this.heaterState.desired_temp++;
      this.setDesiredTemp();
    },
    makeColder: function() {
      this.heaterState.desired_temp--;
      this.setDesiredTemp();
    },
    toggleHeater: function(result) {
      console.log("I was toggled", result);
      if (result.value) {
        this.enableHeater();
      } else {
        this.disableHeater();
      }
    },
    enableHeater: function() {
      axios.post(root + "/enable").then(result => {
        this.loaded = true;
        this.heaterState = result.data;
        console.log("enabled heater", result);
      });
    },
    disableHeater: function() {
      axios.post(root + "/disable").then(result => {
        this.loaded = true;
        this.heaterState = result.data;
        console.log("disabled heater", result);
      });
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 0px;
}
h1 {
  background-color: orangered;
  margin: 0;
}
body {
  margin: 0;
}
#currentTemp {
  font-size: 8em;
}
#desiredTemp {
  font-size: 3em;
  color: lightgray;
}
.adjustTemp {
  margin: 1% 2em;
  font-size: 2.2em;
  -webkit-user-select: none; /* webkit (safari, chrome) browsers */
  -moz-user-select: none; /* mozilla browsers */
  -khtml-user-select: none; /* webkit (konqueror) browsers */
  -ms-user-select: none; /* IE10+ */
  color: lightgray;
}
</style>
