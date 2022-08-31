<template>
  <v-app class="app">
      <button class="button-theme" @click="() => switchTheme(!theme)">
        <SunSvg :class="'sun-svg'"/>
      </button>
      <div class="alert" v-if="!resolution">
        <v-alert
          dense
          outlined
          type="error"
        >
          Разрешите доступ к местоположению!
        </v-alert>
      </div>
    <main class="main"> 
      
      <div class="container">
        <v-autocomplete 
          class="input"
          v-model="input"
          :items="city" 
          :dark="theme"
          @change="selectCity"
        >

        </v-autocomplete>
        <yandex-map 
          :settings="settings"
          :coords="coords"
          zoom="10"
          style="width: 100%; height: 400px"
        >
          <ymap-marker 
            marker-id="123" 
            :coords="coords"
          />
        </yandex-map>
        <v-btn class="button-geo" @click="definitionPosition">
          Где я? 
        </v-btn>
      </div>
    </main>
  </v-app>
</template>

<script>
import { yandexMap, ymapMarker } from 'vue-yandex-maps';
import dataCity from './city.json';
import SunSvg from './components/SunSvg';
import axios from 'axios';

const apiKeyYandex = '5f6e7a79-975f-48f8-8438-b45713ff6a84';

export default {
  name: 'App', 
  components: { yandexMap, ymapMarker, SunSvg },
  data: () => ({
    city: null,
    input: 'Москва',
    coords: ["55.7538789","37.6203735"],
    data: null,
    apiKey: apiKeyYandex,
    settings: {
      apiKey: apiKeyYandex,
      lang: 'ru_RU',
      coordorder: 'latlong',
      enterprise: false,
      version: '2.1'
    },
    theme: false,
    resolution: true,
  }),
  mounted() {
    this.data = dataCity.reduce((previous, current) => {
      previous.set(current['Город'] || current['Регион'],[current['Широта'],current['Долгота']]);
      return previous;
    }, 
    new Map());
    this.city = dataCity.map((i) => i['Город'] || i['Регион']);

    const saveInput =  localStorage.getItem('input');
    const saveCord =  localStorage.getItem('coords');
    const saveTheme =  JSON.parse(localStorage.getItem('theme'));

    if (saveInput) this.input = saveInput;
    if (saveCord) this.coords = JSON.parse(saveCord);
    if (saveTheme !== null) this.switchTheme(saveTheme);

    this.definitionPosition();
  },
  methods: {
    selectCity() {
      localStorage.setItem('input', this.input);
      if (this.input) {
        const newCord = this.data.get(this.input);
        this.coords = newCord;
        localStorage.setItem('coords', JSON.stringify(newCord));
      };
    },
    switchTheme(themeOn) {
      const html = document.querySelector('html');
      if (themeOn) {
        html.style.setProperty('--main-color', '#373737');
        html.style.setProperty('--svg-color', '#f4f4f4');
      } else {
        html.style.setProperty('--main-color', '#f4f4f4');
        html.style.setProperty('--svg-color', '#111111');
      };
      localStorage.setItem('theme', themeOn);
      this.theme = themeOn;
    },
    definitionPosition() {
      this.resolution = true;
      navigator.geolocation.getCurrentPosition((position) => {
      const newCord = [position.coords.latitude, position.coords.longitude];
      this.coords = newCord;
      localStorage.setItem('coords', JSON.stringify(newCord));

      axios
        .get('https://geocode-maps.yandex.ru/1.x/', {
          params: {
            'apikey': this.apiKey,
            'geocode': `${newCord[1]},${newCord[0]}`,
            'kind': 'locality',
            'format': 'json',
            'results': 1,
          }
        })
        .then(r => {this.input = (r.data.response.GeoObjectCollection.featureMember[0].GeoObject.name)});
      },
      () => this.resolution = false
      );
    },
  }
};
</script>

<style lang="css">

  :root {
    --main-color: #f4f4f4;
    --svg-color: #111111;
  }

  html {
    overflow: hidden;
  }

  .app {
    width: 100%;
    height: 100%;
  }

  .alert {
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translateX(-50%);
  }

  .button-theme {
    position: absolute;
    right: 16px;
    top: 16px;
    width: 24px;
    height: 24px;
  }

  .sun-svg {
    fill: var(--svg-color);
  }

  .main {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: var(--main-color);
    transition: background-color 0.3s ease;
  }

  .container {
    max-width: 800px;
    width: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .input {
    max-width: 500px;
    width: 100%;
    margin-bottom: 50px;
  }

  .button-geo {
    margin-top: 16px;
  }

</style>

