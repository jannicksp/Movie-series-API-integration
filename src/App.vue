<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <MovieCollection />
  </div>
</template>

<script>
import MovieCollection from './components/MovieCollection.vue'

export default {
  name: 'App',
  components: {
    MovieCollection
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
