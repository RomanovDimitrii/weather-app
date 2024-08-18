<template>
  <div class="page-background">
    <div class="wrapper">
      <h1>Погодное приложение</h1>
      <p>Узнать погоду в {{ city == '' ? 'вашем городе' : cityName }}</p>

      <form @submit.prevent="getWeather">
        <input placeholder="Введите город" type="text" v-model="city" />
        <button type="submit" v-if="city != ''">Получить погоду</button>
        <button type="submit" v-else disabled>Введите Город</button>
      </form>

      <p class="error">{{ error }}</p>

      <div v-if="info != null" class="weather-info">
        <img v-if="imageUrl" :src="imageUrl" alt="City Image" class="city-image" />
        <div class="weather-details">
          <p>{{ showTemp }}</p>
          <p>{{ showFeelsLike }}</p>
          <p>{{ showMinTemp }}</p>
          <p>{{ showMaxTemp }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { transliterate as tr } from 'transliteration';

export default {
  data() {
    return {
      city: '',
      error: '',
      info: null,
      imageUrl: '' // Сохраняем URL изображения здесь
    };
  },
  computed: {
    cityName() {
      return "'" + this.city + "'";
    },
    cityInLatin() {
      return tr(this.city);
    },
    showTemp() {
      return `Температура сейчас: ${this.info.main.temp} °C`;
    },
    showFeelsLike() {
      return `Ощущается как: ${this.info.main.feels_like} °C`;
    },
    showMinTemp() {
      return `Минимальная температура за сегодня: ${this.info.main.temp_min} °C`;
    },
    showMaxTemp() {
      return `Максимальная температура за сегодня: ${this.info.main.temp_max} °C`;
    }
  },
  methods: {
    getWeather() {
      if (this.city.trim().length < 2) {
        this.error = 'Некорректное название города';
        return false;
      }
      this.error = '';

      const cityInLatin = this.cityInLatin;

      axios
        .get(
          `https://api.openweathermap.org/data/2.5/weather?q=${cityInLatin}&units=metric&appid=9680652078d4b91abd97ef0fa8a56302`
        )
        .then(res => {
          this.info = res.data;
          this.fetchCityImage(cityInLatin);
        })
        .catch(() => {
          this.error = 'Город не найден. Попробуйте еще раз. Подключите VPN';
          this.info = null;
        });
    },
    fetchCityImage(city) {
      const apiKey = '45507249-b1448bebacb5be056e5d59560'; // Ваш API ключ Pixabay
      axios
        .get(`https://pixabay.com/api/?key=${apiKey}&q=${city}&image_type=photo`)
        .then(response => {
          if (response.data.hits.length > 0) {
            this.imageUrl = response.data.hits[0].webformatURL;
          } else {
            this.imageUrl = ''; // Если изображение не найдено
          }
        })
        .catch(error => {
          console.error('Ошибка при загрузке изображения:', error);
          this.imageUrl = ''; // Если произошла ошибка
        });
    }
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

/* Фон для всей страницы */
.page-background {
  /* background: linear-gradient(to right, #4facfe 0%, #00f2fe 100%); */
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

/* Основной контейнер */
.wrapper {
  max-width: 900px;
  width: 100%;
  margin: 50px auto;
  border-radius: 20px;
  padding: 40px;
  background: rgba(0, 0, 0, 0.7); /* Полупрозрачный темный фон */
  text-align: center;
  color: white;
  font-family: 'Roboto', sans-serif;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.5);
}

.wrapper h1 {
  font-size: 2.5em;
  margin-bottom: 20px;
}

.wrapper p {
  font-size: 1.2em;
  margin-top: 20px;
}

.error {
  color: #d03939;
  font-weight: bold;
  margin-top: 10px;
}

.wrapper input {
  margin-top: 30px;
  background: rgba(255, 255, 255, 0.1);
  border: 0;
  border-bottom: 2px solid #110813;
  color: #fcfcfc;
  font-size: 18px;
  padding: 10px 15px;
  outline: none;
  width: 80%;
  max-width: 500px;
  transition: border-color 0.3s ease;
}

.wrapper input:focus {
  border-bottom-color: #6e2d7d;
}

.wrapper button {
  background: #e3bc4b;
  color: #fff;
  border-radius: 10px;
  border: 2px solid #b99935;
  padding: 10px 20px;
  margin-top: 30px;
  cursor: pointer;
  font-size: 16px;
  transition: transform 0.3s ease, background-color 0.3s ease;
}

.wrapper button:hover {
  transform: scale(1.05) translateY(-3px);
  background-color: #d1a343;
}

.wrapper button:disabled {
  background: #746027;
  cursor: not-allowed;
}

.weather-info {
  margin-top: 40px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.city-image {
  width: 100%;
  max-width: 600px;
  border-radius: 20px;
  margin-bottom: 20px;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
}

.weather-details p {
  margin: 10px 0;
  font-size: 1.2em; /* Увеличенный размер шрифта для данных о погоде */
}
</style>
