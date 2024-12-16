<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        placeholder="Search"
        dark
        borderless
        @keyup.enter="getWetherBySearch"
      >
        <template v-slot:before>
          <q-icon @click="getLocation()" name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="getWetherBySearch" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>

        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>

        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>

          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img
          :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
          alt="test"
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar <br />
          Weather
        </div>

        <q-btn class="col" flat @click="getLocation()">
          <q-icon left size="3em" name="my_location" />
          <div>Find my Location</div>
        </q-btn>
      </div>
    </template>

    <div class="col skyline"></div>
  </q-page>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue';
import { useQuasar } from 'quasar';
import { api as $axios } from 'boot/axios';

interface IWeatherData {
  name: string;
  weather: Array<{
    main: string;
    icon: string;
  }>;
  main: {
    temp: number;
  };
}

const $q = useQuasar();
const search = ref('');
const weatherData = ref<IWeatherData | null>(null);
const lat = ref<number | null>(null);
const lon = ref<number | null>(null);
const apiUrl = 'https://api.openweathermap.org/data/2.5/weather';
const apiKey = 'fb7b8afaec8198ad92af6e1ebd38d085';

const bgClass = computed(() => {
  if (weatherData.value) {
    if (weatherData.value.weather[0].icon.endsWith('n')) {
      return 'bg-night';
    } else {
      return 'bg-day';
    }
  }

  return 'bg-day';
});

const getLocation = () => {
  navigator.geolocation.getCurrentPosition((position) => {
    lat.value = position.coords.latitude;
    lon.value = position.coords.longitude;
    getWetherByCoords();
  });
};

const getWetherByCoords = async () => {
  try {
    // Показываем индикатор загрузки
    $q.loading.show();

    // Формируем URL для запроса
    const weatherUrl = `${apiUrl}?APPID=${apiKey}&lat=${lat.value}&lon=${lon.value}&units=metric`;

    // Получаем данные о погоде
    const response = await $axios(weatherUrl);
    search.value = response.data.name;
    weatherData.value = response.data;
  } catch (error) {
    // Обработка ошибки получения данных
    $q.notify({
      message: 'Ошибка при получении данных о погоде',
      color: 'negative',
      position: 'top',
    });
  } finally {
    // Скрываем индикатор загрузки
    $q.loading.hide();
  }
};

const getWetherBySearch = async () => {
  // Проверка наличия введенного города
  const isCityEntered = search.value.length > 0;

  if (!isCityEntered) {
    $q.notify({
      message: 'Пожалуйста, введите название города',
      color: 'red',
      position: 'top',
      timeout: 1000,
    });
    return;
  }

  // Показываем индикатор загрузки
  $q.loading.show();

  try {
    // Формируем URL для запроса
    const weatherUrl = `${apiUrl}?APPID=${apiKey}&q=${search.value}&units=metric`;

    // Получаем данные о погоде
    const response = await $axios(weatherUrl);
    console.log(response);
    weatherData.value = response.data;
  } catch (error) {
    // Обработка ошибки получения данных
    $q.notify({
      message: 'Ошибка при получении данных о погоде',
      color: 'negative',
      position: 'top',
    });
  } finally {
    // Скрываем индикатор загрузки
    $q.loading.hide();
  }
};
</script>

<style lang="scss">
.q-page {
  $gradient-start: #136a8a;
  $gradient-end: #267871;

  background: linear-gradient(to bottom, $gradient-start, $gradient-end);

  &.bg-night {
    $night-start: #232526;
    $night-end: #414345;

    background: linear-gradient(to right, $night-start, $night-end);
  }

  &.bg-day {
    $day-start: #00b4db;
    $day-end: #0083b0;

    background: linear-gradient(to right, $day-start, $day-end);
  }
}

.degree {
  top: -44px;
}

.skyline {
  $skyline-height: 100px;

  flex: 0 0 $skyline-height;
  background: {
    image: url(../statics/skyline.png);
    size: contain;
    position: center bottom;
  }
}
</style>
