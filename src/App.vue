<template>
  <div align="center" id="app">
    <PlayingField v-bind:gameIsStarted="gameIsStarted" :field="field" />

    <div v-if="this.game.playingFieldSize < 1">
      <h2>Добрейший вечерочек</h2>
      <h4>Введите размер игрового поля</h4>от 20 до 50 клеток:
      <label>
        <input v-model.lazy.number.trim="inputFieldSize" />
      </label>
    </div>

    <ControlButtons v-bind:startGame="startGame" v-bind:stopGame="stopGame" v-if="this.field" />
  </div>
</template>

<script>
import PlayingField from "./components/app/PlayingField.vue";
import ControlButtons from "./components/app/ControlButtons.vue";
import axios from "axios";

export default {
  name: "app",
  components: {
    PlayingField,
    ControlButtons
  },
  data: function() {
    return {
      field: [], //игровое поле
      inputFieldSize: null, //размер игрового поля, введенный пользователем
      gameIsStarted: false, //проверка на запуск игры
      game: {
        playingFieldSize: 0,
        cells: []
      }
    };
  },
  methods: {
    startGame: function() {
      this.gameIsStarted = true;

      this.game.cells = [];
      this.field.forEach(row => {
        row.forEach(cell => {
          if (cell.life) this.game.cells.push(cell);
        });
      });

      axios
        .post("http://localhost:8075/game", this.game)
        .then(response => {
          this.game = response.data;

          this.field = this.newFields(this.game);

          //При условии, что игра не остановлена, вызывается startGame()
          if (this.gameIsStarted && this.game.cells.length > 0) {
            this.game.cells = [];
            setTimeout(this.startGame, 10);
            this.game.cells = [];
          }

        })
        .catch(error => {
          /* eslint-disable no-console */
          console.log(error);
          /* eslint-enable no-console */
          alert(
            "При ожидании данных с сервера что-то пошло не так, как должно..."
          );
        });
    },
    //Остановка игры
    stopGame: function() {
      this.gameIsStarted = false;
    },

    //Функция для создания новых таблиц
    newFields: function(newGame) {
      let newGenerationField = [];
      for (let y = 1; y <= newGame.playingFieldSize; y++) {
        newGenerationField.push([]);
        for (let x = 1; x <= newGame.playingFieldSize; x++) {
          newGenerationField[y - 1].push({
            y: y,
            x: x,
            life: false
          });
        }
      }

      newGame.cells.forEach(cell => {
        newGenerationField[cell.y - 1][cell.x - 1].life = true;
      });

      this.field = newGenerationField;
      return newGenerationField;
    }
  },

  watch: {
    inputFieldSize: function(val) {
      val = parseInt(val, 10);
      if (val > 19 && val < 51) {
        this.game.playingFieldSize = val;

        this.field = this.newFields(this.game);
      } else {
        alert("Введите значение от 20 до 50");
      }
    }
  },
  //при закрытии или обновлении браузера с сервера бурутся последние данные и формируется игровое поле
  created: function() {
    axios.get("http://localhost:8075/game").then(response => {
      if (response.data !== "") {
        this.game = response.data;

        this.field = this.newFields(this.game);

        this.startGame();
      }
    });
  }
};
</script>
