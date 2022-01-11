<template>
  <div id="app">
    <Learn msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import Learn from './views/Index/Animation.vue'

export default {
  name: 'App',
  components: {
    Learn
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
