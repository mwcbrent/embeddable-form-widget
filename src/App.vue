<template>
  <div v-if="loaded">
    <div v-if="submitted">Successfully submitted!</div>
    <vue-form-generator v-else="!submitted" :schema="schema" :model="model" :options="formOptions"></vue-form-generator>
  </div>
</template>

<script>
  import Vue from 'vue'
  import VueFormGenerator from 'vue-form-generator/dist/vfg-core.js'
  import 'vue-form-generator/dist/vfg-core.css'

  Vue.use(VueFormGenerator)

  const widgetName = 'formWidget';

  export default {
    name: 'app',

    data() {

      return {
        apiKey: null,
        useAuthHeader: null,
        config: null,
        model: null,
        schema: null,
        formOptions: null,
        loaded: false,
        submitted: false
      }
    },


    created() {

      // process any config passed along with the snippet
      this.processConfig();

      // fetch the form schema
      this.fetchSchema();
    },


    computed: {

      /**
       * Generates the proper schema url depending on auth options.
       * @return {[type]} [description]
       */
      schemaUrl() {
        const url = window[widgetName].schemaUrl;
        if (!this.useAuthHeader) {
          return `${url}?api_key=${this.apiKey}`
        }
        return url
      }

    },


    methods: {

      /**
       * Loads the form schema
       * @return {[type]} [description]
       */
      fetchSchema() {

        // use auth header if this option is set
        const options = {}

        if (this.useAuthHeader) {
          options.headers = {
            authorization: this.apiKey
          }
        }

        // load the schema
        fetch(this.schemaUrl, options)
          .then((response) => response.json())
          .then((json) => {

            // set config
            this.config = json.config

            // assign the schema and model
            this.model = json.model
            this.schema = this.processSchema(json.schema)
            this.formOptions = json.formOptions

            // indicate that the schema has been loaded
            this.loaded = true
          })
          .catch((ex) => {
            // TODO: error handling
          })
      },


      /**
       * Processes any configuration attached to the window.
       * @return {[type]} [description]
       */
      processConfig() {

        // grab the api key
        this.apiKey = window[widgetName].apiKey

        // determine whether to use auth header or not
        this.useAuthHeader = window[widgetName].useAuthHeader
      },


      /**
       * Adds validators and additional fields to form schema.
       * @param  {[type]} schema [description]
       * @return {[type]}        [description]
       */
      processSchema(schema) {

        // map validators to real functions
        schema.fields.map((field) => {

          if (field.validator) {
            field.validator = VueFormGenerator.validators[field.validator]
          }

          return field
        })

        // add submit button to the schema
        const submitButton = {
          type: 'submit',
          validateBeforeSubmit: true,
          onSubmit: this.submit
        }

        schema.fields.push(submitButton)

        return schema;
      },


      /**
       * Posts data from the form on submit.
       * @param  {[type]} data [description]
       * @return {[type]}      [description]
       */
      submit (data) {

        // base request options
        const options = {
          method: 'POST',
          body: JSON.stringify(data)
        }

        if (this.useAuthHeader) {
          options.headers = {
            authorization: this.apiKey
          }
        }

        // post that bitch
        fetch(this.config.postUrl, options)
          .then((response) => {

            if (response.status === 201) {
              this.submitted = true
            }

          })
      }
    }
  }
</script>

<style scoped>
</style>
