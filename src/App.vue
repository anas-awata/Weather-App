<template>
  <div id="app" :class="weatherstatus" v-cloak>
    <main>
      <div class="searchbox">
        <input
          type="text"
          v-model="query"
          placeholder="Search.."
          class="searchbar"
          @keypress="fetchweather"
        />
        <img @click="mountlocator" src="./assets/location.png" alt="locate" />
      </div>
      <div class="weatherwrap" v-if="typeof weather.main != 'undefined'">
        <button @click="changefeh" :disabled="disabledF">°F</button>
        <button @click="changece" :disabled="disabledC">°C</button>
        <div class="locationbox">
          <div class="location">
            {{ weather.name }},{{ weather.sys.country }}
          </div>
          <div class="date">{{ datebuilder() }}</div>
        </div>
        <div class="weatherbox">
          <div class="temp">
            {{ Math.round(weather.main.temp) + unitsymbol }}
          </div>
          <div class="feels">
            Feels Like: {{ Math.round(weather.main.feels_like) + unitsymbol }}
          </div>
          <div class="status">{{ weather.weather[0].main }}</div>
        </div>
      </div>
      <div v-else class="notfound">
        Location Not Found, Try searchig another location
      </div>
    </main>
  </div>
</template>

<script>
export default {
  name: "App",
  components: {},
  data() {
    return {
      apikey: "023e3f41fd38a58d25d9d98cb1a9e402",
      url_base: "https://api.openweathermap.org/data/2.5/",
      query: "",
      weather: {},
      unit: "metric",
      unitsymbol: "°C",
      disabledF: false,
      disabledC: true,
      lat: "",
      lon: "",
      Time: "",
    };
  },
  methods: {
    //create the fetch weather API method for user location on mount.
    mountfetch() {
      fetch(
        `${this.url_base}weather?lat=${this.lat}&lon=${this.lon}&units=${this.unit}&APPID=${this.apikey}`
      )
        .then((res) => {
          return res.json();
        })
        .then(this.setresults);
    },
    //create the fetch weather API method for user input
    queryfetch() {
      fetch(
        `${this.url_base}weather?q=${this.query}&units=${this.unit}&APPID=${this.apikey}`
      )
        .then((res) => {
          return res.json();
        })
        .then(this.setresults);
    },
    //fetching the weather After writing city name and hitting Enter.
    fetchweather(e) {
      if (e.key == "Enter") {
        this.unit = "metric";
        this.unitsymbol = "°C";
        this.queryfetch();

        //setting the default value of button for C every time.
        if (this.unit == "imperial") {
          this.disabledF = true;
          this.disabledC = false;
        } else {
          this.disabledF = false;
          this.disabledC = true;
        }
      }
    },
    //storing the result in weather data.
    //and changing the name in input box.
    setresults(results) {
      this.weather = results;
      this.query = this.weather.name;
    },
    //function to get the Date formatted.
    datebuilder() {
      let d = new Date();
      let months = [
        "Jan",
        "Feb",
        "Mar",
        "Apr",
        "May",
        "Jun",
        "Jul",
        "Aug",
        "sep",
        "Oct",
        "Nov",
        "Dec ",
      ];
      let days = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ];
      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();
      this.Time = d.getHours();
      return `${day} ${date} ${month} ${year}`;
    },
    //fetching new data for imperial system.
    changefeh() {
      this.unit = "imperial";
      this.queryfetch();
      //changing the apeerance to fit the imperial system.
      this.unitsymbol = "°F";
      this.disabledF = !this.disabledF;
      this.disabledC = !this.disabledC;
    },
    //fetching new data for metric system.
    changece() {
      this.unit = "metric";
      this.queryfetch();
      //changing the apeerance to fit the imperial system.
      this.unitsymbol = "°C";
      this.disabledF = !this.disabledF;
      this.disabledC = !this.disabledC;
    },
    //geolocator method to get user location then mount it.
    mountlocator() {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          this.lat = position.coords.latitude;
          this.lon = position.coords.longitude;
          this.mountfetch();
        },
        (error) => {
          console.log(error.message);
        }
      );
    },
  },
  computed: {
    //changing the app class to change background at each weather status.
    weatherstatus() {
      if (
        typeof this.weather.main != "undefined" &&
        this.weather.weather[0].main == "Clouds" &&
        this.Time > 6 &&
        this.Time < 19
      ) {
        return "clouds";
      } else if (
        typeof this.weather.main != "undefined" &&
        this.weather.weather[0].main == "Clouds" &&
        (this.Time < 6 || this.Time > 19)
      ) {
        return "cloudy-night";
      } else if (
        typeof this.weather.main != "undefined" &&
        this.weather.weather[0].main == "Clear" &&
        this.Time > 6 &&
        this.Time < 19
      ) {
        return "warm";
      } else if (
        typeof this.weather.main != "undefined" &&
        this.weather.weather[0].main == "Clear" &&
        (this.Time < 6 || this.Time > 19)
      ) {
        return "clear-night";
      } else if (
        typeof this.weather.main != "undefined" &&
        this.weather.weather[0].main == "Rain"
      ) {
        return "rain";
      } else if (
        (typeof this.weather.main != "undefined" &&
          this.weather.main.temp < 0 &&
          this.unit == "metric") ||
        (typeof this.weather.main != "undefined" &&
          this.weather.main.temp < 32 &&
          this.unit == "imperial")
      ) {
        return "winter";
      } else {
        return "";
      }
    },
  },
  //fetching the Data for user location at mounted.
  mounted() {
    this.mountlocator();
  },
};
</script>
