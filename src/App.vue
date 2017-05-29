<template>
  <div id="app" v-if="loaded">
    <vue-form-generator :schema="schema" :model="model" :options="formOptions"></vue-form-generator>
  </div>
</template>

<script>
  import Vue from 'vue'
  import VueFormGenerator from 'vue-form-generator/dist/vfg-core.js'
  import 'vue-form-generator/dist/vfg-core.css'

  Vue.use(VueFormGenerator)

  export default {
    name: 'app',

    data: function () {
      return {
        loaded: false,
        model: null,
        schema: null,
        formOptions: null
      }
    },

    created: function () {
      return fetch('http://localhost:3000/forms')
        .then((response) => response.json())
        .then((json) => {
          this.loaded = true
          this.model = json.model
          this.schema = json.schema
          this.formOptions = json.formOptions
        })
        .catch((ex) => {
          console.log('parsing failed', ex)
        })
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
