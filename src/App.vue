<template>
  <div id="app" v-if="loaded">
    <div v-if="submitted">Successfully submitted!</div>
    <vue-form-generator v-else="!submitted" :schema="schema" :model="model" :options="formOptions"></vue-form-generator>
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
        model: null,
        schema: null,
        formOptions: null,
        apiKey: null,
        loaded: false,
        submitted: false
      }
    },

    created: function () {

      const options = {}

      // use auth header if this option is set
      if (window.formWidget.apiKey) {
        options.headers = {
          authorization: window.formWidget.apiKey
        }
      }

      fetch('http://localhost:3000/forms', options)
        .then((response) => response.json())
        .then((json) => {

          json.schema.fields.map((field) => {
            // map submit button to local function
            if (field.type === 'submit') {
              field.onSubmit = this.submit
            }

            // map validators to real functions
            if (field.validator) {
              field.validator = VueFormGenerator.validators[field.validator]
            }

            return field
          })

          // assign the schema and model
          this.model = json.model
          this.schema = json.schema
          this.formOptions = json.formOptions

          // indicate that the schema has been loaded
          this.loaded = true
        })
        .catch((ex) => {
          // TODO: error handling
        })
    },

    methods: {

      submit (data) {

        // base request options
        const options = {
          headers: {
            'content-type': 'application/json'
          },
          method: 'POST',
          body: JSON.stringify(data)
        }

        // use auth header if this option is set
        if (window.formWidget.apiKey) {
          options.headers.authorization = window.formWidget.apiKey
        }

        // post that bitch
        fetch('http://localhost:3000/form', options)
          .then((response) => {

            if (response.status === 201) {
              this.submitted = true
            }

            // TODO: error handling
          })
      }
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
