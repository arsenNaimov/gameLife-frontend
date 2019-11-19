<template>
     <div>
        <input type="button" value="Начать игру" @click="startGame"/>
        <input type="button" value="Остановить игру" @click="stopGame"/>
    </div>
</template>

<script>
export default {
        props: ['startGame', 'stopGame']
    
}
</script>

<style scoped>

</style>