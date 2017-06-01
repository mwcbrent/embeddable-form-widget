<template>
  <div id="jpjd-formwidget--app">
    <!--  display spinner if loading schema or submitting form -->
    <spinner v-if="loading" />

    <!--  form loading, but not submitted -->
    <vue-form-generator v-else-if="!loading && !submitted" :schema="schema" :model="model" :options="formOptions"></vue-form-generator>

    <!--  form submitted successfully -->
    <div v-else-if="submitted">
      Thanks!
    </div>
  </div>
</template>

<script>
  import Vue from 'vue'
  import Spinner from './components/Spinner';
  import VueFormGenerator from 'vue-form-generator/dist/vfg-core.js'
  import 'vue-form-generator/dist/vfg-core.css'

  const widgetName = 'formWidget'

  export default {
    name: 'app',

    components: {
      Spinner,
      'vue-form-generator': VueFormGenerator.component
    },

    data() {

      return {
        config: null,
        model: {},
        schema: null,
        formOptions: null,
        loading: true,
        submitted: false
      }
    },


    created() {

      // get window config
      this.config = window[widgetName]

      // set api key on model if not using auth header
      if (!this.config.useAuthHeader) {
        this.model.api_token = this.config.apiKey
      }

      // fetch the form schema
      this.fetchSchema()
    },


    computed: {

      /**
       * Generates the proper schema url depending on auth options.
       * @return {[type]} [description]
       */
      schemaUrl() {

        const url = this.config.schemaUrl

        if (!this.config.useAuthHeader) {
          return `${url}?api_token=${this.config.apiKey}`
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

        const options = {}

        // use auth header if this option is set
        if (this.config.useAuthHeader) {
          options.headers = {
            authorization: this.config.apiKey
          }
        }

        // load the schema
        fetch(this.schemaUrl, options)
          .then((response) => response.json())
          .then((json) => {

            // set config
            this.config.postUrl = (json.config && json.config.postUrl)

            // assign the schema and form options
            this.schema = this.processSchema(json.schema)
            this.formOptions = json.formOptions
          })
          .catch((err) => {
            // TODO: add error handling
            console.log("[fetch] error")
            console.log(err)
          })
          .then(() => {
            // disable loading spinner
            this.loading = false;
          })
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

        return schema
      },


      /**
       * Posts data from the form on submit.
       * @param  {[type]} data [description]
       * @return {[type]}      [description]
       */
      submit (data) {

        // trigger loading spinner
        this.loading = true;

        // base request options
        const options = {
          headers: {
            'content-type': 'application/json'
          },
          method: 'POST',
          body: JSON.stringify(data)
        }

        // use auth header if this option is set
        if (this.config.useAuthHeader) {
          options.headers.authorization = this.config.apiKey
        }

        // post that bitch
        fetch(this.config.postUrl, options)
          .then((response) => {

            if (response.status === 201) {
              this.submitted = true
            }

          })
          .catch((err) => {
            // TODO: add error handling
          })
          .then(() => {
            // disable loading spinner
            this.loading = false;
          })
      }
    }
  }
</script>

<style>
  #jpjd-formwidget--app {
    display: flex;
    justify-content: center;
    min-height: 300px;
  }

  fieldset {
    border: 0;
  }
</style>
