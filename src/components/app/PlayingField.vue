<template>
<table cellspacing="0">
                <tr v-for="column in field" v-bind:key="column.id">
                    <td :class="{ life: cell.life }" v-for="cell in column" :key="cell.id"  @mouseover="cell.life = !cell.life"></td>
                </tr>
</table>
</template>

<script>
export default {
    props: ['gameIsStarted', 'game', 'field'],
}
</script>

<style scoped>
td {
  border: solid 1px black;
  width: 10px;
  height: 10px;
}
.life{
    background-color: black;
}
</style>