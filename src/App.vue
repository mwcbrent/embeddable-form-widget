<template>
  <div id="app">
    <widget></widget>
  </div>
</template>

<script>
import Widget from './components/Widget'

export default {
  name: 'app',
  components: {
    Widget
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
