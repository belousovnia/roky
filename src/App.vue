<template>
  <v-app class="app">
    <main class="main"> 
      <div class="container">
        <v-autocomplete 
          class="input"
          v-model="input"
          :items="city"
        >

        </v-autocomplete>
        <yandex-map 
          :settings="settings"
          :coords="cord"
          zoom="10"
          style="width: 100%; height: 400px"
        >
          <ymap-marker 
            marker-id="123" 
            :coords="cord"
          />
        </yandex-map>
      </div>
    </main>
  </v-app>
</template>

<script>
import { yandexMap, ymapMarker } from 'vue-yandex-maps';
import dataCity from './city.json'

export default {
  name: 'App', 
  components: { yandexMap, ymapMarker },
  data: () => ({
    city: null,
    input: null,
    cord: ["55.7538789","37.6203735"],
    data: null,
    settings: {
      apiKey: '5f6e7a79-975f-48f8-8438-b45713ff6a84',
      lang: 'ru_RU',
      coordorder: 'latlong',
      enterprise: false,
      version: '2.1'
    },
  }),
  mounted() {
    this.data = dataCity.reduce((previous, current) => {
      previous.set(current['Город'] || current['Регион'],[current['Широта'],current['Долгота']]);
      return previous;
    }, 
    new Map());
    this.city = dataCity.map((i) => i['Город'] || i['Регион']);

    const saveInput =  localStorage.getItem('input');
    const saveCord =  localStorage.getItem('cord');
    
    if (saveInput) this.input = saveInput;
    if (saveCord) this.cord = JSON.parse(saveCord);
  },
  watch: {
    input(i) {
      localStorage.setItem('input', i);
      if (i) {
        const newCord = this.data.get(i);
        this.cord = newCord;
        localStorage.setItem('cord', JSON.stringify(newCord));
      };
    }
  },
};
</script>

<style lang="css">
  .app {
    width: 100%;
    height: 100%;
  }

  .main {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
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
</style>

