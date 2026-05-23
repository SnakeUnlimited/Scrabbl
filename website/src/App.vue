<template>
  
  <EntryPoint msg="test"/>
</template>

<script>
import EntryPoint from './components/GameScreen.vue'
//<img alt="Vue logo" src="./assets/logo.png">
export default {
  name: 'App',
  components: {
    EntryPoint
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
