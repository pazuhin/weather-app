<template>
  <div class="text-white mb-8">
    <div class="places-input text-grey-800">
      <input type="search" id="address" class="form-control" placeholder="Where are we going?" />
      <p>Selected: <strong id="address-value">none</strong></p>
    </div>
    <div class="weather-container font-sans w-128 max-w-lg rounded-lg overflow-hidden bg-gray-900 shadow-lg mt-4">
      <div class="current-weather flex items-center justify-between px-6 py-8">
        <div class="flex items-center">
          <div>
            <div class="text-6xl font-semibold">{{ currentTemperature.actual }}°C</div>
            <div>Feels like {{ currentTemperature.feels }}°C</div>
          </div>
          <div class="mx-5">
            <div class="font-semobold">{{ currentTemperature.summary }}</div>
            <div>{{ location.name }}</div>
          </div>
        </div>
        <div style="margin-right: -25px"><img :src="currentTemperature.icon" /></div>
      </div> <!--      end current weather-->

      <div class="feature-weather text-sm bg-grey-800 px-6 py-8 overflow-hidden">
        <div
          v-for="(day, index) in daily"
          :key="day.time"
          class="flex items-center"
          :class="{'mt-8' : index > 0}"
          v-if="index < 5"
        >
          <div class="w-1/6 text-lg text-gray-200">{{ toDayOfWeek(day.dt) }}</div>
          <div class="w-4/6 px-4 flex items-center">
            <div><img :src=correctIconPath(day.weather[0].icon) alt=""></div>
            <div class="ml-3">{{ day.weather[0].main }}</div>
          </div>
          <div class="w-1/6 text-right">
            <div>{{ convertTocelsius(day.temp.max) }}°C</div>
            <div>{{ convertTocelsius(day.temp.min) }}°C</div>
          </div>
        </div>
      </div>  <!--      end feature weather-->
    </div> <!--      end weather-container weather-->
  </div>
</template>

<script>
  export default {
    mounted() {
      this.fetchData()

      var placesAutocomplete = places({
        appId: 'plIB0NS47Q2P',
        apiKey: 'a3ee1fb52d7f8b7844f1ea06f899a161',
        container: document.querySelector('#address')
      }).configure({
        type: 'city',
        aroundLatLngViaIP: false,
      });

      var $address = document.querySelector('#address-value')
      placesAutocomplete.on('change', e =>  {
        $address.textContent = e.suggestion.value
        this.location.name = `${e.suggestion.name}, ${e.suggestion.country}`
        this.location.lat = e.suggestion.latlng.lat
        this.location.lon = e.suggestion.latlng.lng
      });

      placesAutocomplete.on('clear', function() {
        $address.textContent = 'none';
      });
    },
    watch: {
      location: {
        handler(newValue, oldValue) {
          this.fetchData()
        },
        deep: true
      }
    },
    data() {
      return {
        currentTemperature: {
          actual: '',
          feels: '',
          summary: '',
          icon: ''
        },
        daily: [],
        location: {
          name: 'Toronto, Canada',
          id: 6167865,
          lat: 55.752,
          lon: 37.616
        }
      }
    },
    methods: {
      fetchData() {
        fetch(`/api/weather/?lat=${this.location.lat}&lon=${this.location.lon}`)
        .then(response => response.json())
        .then(data => {
          console.log(data)

          this.currentTemperature.actual = this.convertTocelsius(data.current.temp);
          this.currentTemperature.feels = this.convertTocelsius(data.current.feels_like);
          this.currentTemperature.icon = "http://openweathermap.org/img/wn/" + data.current.weather[0].icon + "@2x.png";
          this.currentTemperature.summary = data.current.weather[0].main;
          this.daily = data.daily;
        })
      },
      convertTocelsius(kelvin) {
        return parseFloat(kelvin - 273.15).toFixed(0)
      },
      toDayOfWeek(timestamp) {
        const newDate = new Date(timestamp*1000);
        const days = ['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT'];

        return days[newDate.getDay()];
      },
      correctIconPath(icon) {
        return "http://openweathermap.org/img/wn/" + icon + ".png";
      }
    },
  }
</script>
