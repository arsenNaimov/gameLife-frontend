<template>
  <div align="center" id="app">
    <PlayingField v-bind:game="game" v-bind:gameIsStarted="gameIsStarted" />

    <div v-if="this.game.playingFieldSize < 1">
      <h2>Добрейший вечерочек</h2>
      <h4>Введите размер игрового поля</h4>от 20 до 50 клеток:
      <label>
        <input v-model.lazy.number.trim="inputFieldSize" />
      </label>
    </div>

    <ControlButtons
      v-bind:startGame="startGame"
      v-bind:stopGame="stopGame"
      v-if="this.game.playingFieldSize > 9"
    />
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
      inputFieldSize: null, //размер игрового поля, введенный пользователем
      oldGenerationCells: [], //старое поколение ячеек
      newGenerationCells: [], //новое поколение ячеек
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
      for (let i = 0; i < this.game.cells.length; i++) {
        this.game.cells[i].life = true;
      }

      for (let i = 0; i < this.oldGenerationCells.length; i++) {
        document.getElementById(
          this.oldGenerationCells[i].y + "," + this.oldGenerationCells[i].x
        ).bgColor = ""; //чистка поля от мертвых клеток
      }

      this.newGenerationCells = [];
      axios
        .post("http://localhost:8075/game", this.game)
        .then(response => {
          this.game.playingFieldSize = response.data.playingFieldSize;

          for (let i = 0; i < response.data.cells.length; i++) {
            let newGenerationCell = {
              y: response.data.cells[i].y,
              x: response.data.cells[i].x
            };
            this.newGenerationCells.push(newGenerationCell);
          }

          for (let y = 1; y <= this.game.playingFieldSize; y++) {
            for (let x = 1; x <= this.game.playingFieldSize; x++) {
              document.getElementById(y + "," + x).bgColor = "";
            }
          }

          for (var i = 0; i < this.newGenerationCells.length; i++)
            document.getElementById(
              this.newGenerationCells[i].y + "," + this.newGenerationCells[i].x
            ).bgColor = "black";
          this.game.cells = this.newGenerationCells;

          if (this.game.cells.length > 0 && this.gameIsStarted === true)
            setTimeout(this.startGame, 100);
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

    stopGame: function() {
      this.gameIsStarted = false;
      this.newGenerationCells = [];

      for (let y = 1; y <= this.game.playingFieldSize; y++) {
        for (let x = 1; x <= this.game.playingFieldSize; x++) {
          document.getElementById(y + "," + x).bgColor = "";
        }
      }
      this.game.cells = [];
    }
  },

  watch: {
    inputFieldSize: function(val) {
      val = parseInt(val, 10);
      val > 19 && val < 51
        ? (this.game.playingFieldSize = val)
        : alert("Введите значение от 20 до 50");
    }
  },

  created: function() {
    axios.get("http://localhost:8075/game").then(response => {
      if (response.data !== "") {
        this.game.playingFieldSize = response.data.playingFieldSize;
        for (let i = 0; i < response.data.cells.length; i++) {
          let cell = {
            y: response.data.cells[i].y,
            x: response.data.cells[i].x
          };
          this.game.cells.push(cell);
        }
        this.startGame();
      }
    });
  }
};
</script>

