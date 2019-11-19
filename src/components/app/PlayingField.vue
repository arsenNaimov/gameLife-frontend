<template>
<table cellspacing="0">
                <tr v-for="column in game.playingFieldSize" v-bind:key="column.id">
                    <td  class="square" :id="[column,line]" v-for="line in game.playingFieldSize" v-bind:key="line.id"  v-on:mouseover="selectCell"></td>
                </tr>
</table>
</template>

<script>
export default {
     props: ['gameIsStarted', 'game'],
     methods: {
        selectCell: function (event) {
     
            if (this.gameIsStarted === true) {
                return;
            }
            if (document.getElementById(event.target.id).bgColor) {
                document.getElementById(event.target.id).bgColor = "";
                this.game.cells.splice(this.game.cells.indexOf({
                    y: event.target.id.split(",")[0],
                    x: event.target.id.split(",")[1]
                }), 1)
            } else {
                document.getElementById(event.target.id).bgColor = "black";
                this.game.cells.push({
                    y: event.target.id.split(",")[0],
                    x: event.target.id.split(",")[1]
                });
            }

        }
    }
}
</script>

<style scoped>
td.square {
  border: solid 1px black;
  width: 10px;
  height: 10px;
}
</style>