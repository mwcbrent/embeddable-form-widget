<template>
  <div id="jpjd-formwidget--app">
    <!--  display error if anything goes wrong -->
    <alert v-if="error" :msg="error" />

    <!--  display spinner if loading schema or submitting form -->
    <spinner v-if="loading" />

    <!--  form loading, but not submitted -->
    <vue-form-generator v-else-if="!loading && !submitted" :schema="schema" :model="model" :options="formOptions"></vue-form-generator>

    <!--  form submitted successfully -->
    <div class="confirmation" v-else-if="submitted">
      <h2>Success!</h2>
      <Checkmark />
    </div>
  </div>
</template>

<script>
  import Vue from 'vue'
  import Checkmark from './components/Checkmark'
  import Alert from './components/Alert'
  import Spinner from './components/Spinner'
  import VueFormGenerator from "vue-form-generator";
  import 'vue-form-generator/dist/vfg-core.css'
  import 'whatwg-fetch'

  const widgetName = 'formWidget'

  export default {
    name: 'app',

    components: {
      Alert,
      Checkmark,
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
        submitted: false,
        error: null
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
       * Get the value for a given key in address_components
       *
       * @param {Array} components address_components returned from Google maps autocomplete
       * @param type key for desired address component
       * @returns {String} value, if found, for given type (key)
       * @public
       */
      extract(components, type, shortName = false) {
        return components.filter((component) => component.types.indexOf(type) === 0).map((item) => shortName ? item.short_name : item.long_name).pop() || null;
      },

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

            // bind model and assign the schema and form options
            this.model = json.model
            this.schema = this.processSchema(json.schema)
            this.formOptions = json.formOptions

            // reset error state
            this.error = null
          })
          .catch((err) => {
            this.error = 'Sorry! It looks like there was an issue loading the form. Try refreshing the page.'
          })
          .then(() => {
            // disable loading spinner
            this.loading = false
          })
      },


      /**
       * Adds validators and additional fields to form schema.
       * @param  {[type]} schema [description]
       * @return {[type]}        [description]
       */
      processSchema(schema) {

        // map schema to local functions and validators
        schema.fields.map((field) => {

          // map validators to functions
          if (field.validator) {
            field.validator = VueFormGenerator.validators[field.validator]
          }

          // map address field to local function
          if (field.type === 'googleAddress') {
            field.onPlaceChanged = this.placeChanged
          }

          return field
        })

        // add submit button to the schema
        const submitButton = {
          type: 'submit',
          buttonText: 'Submit',
          validateBeforeSubmit: true,
          onSubmit: this.submit
        }

        schema.fields.push(submitButton)

        return schema
      },


      /**
       * Called when address field changes.
       * @param  {[type]} value    [description]
       * @param  {[type]} place    [description]
       * @param  {[type]} rawPlace [description]
       * @param  {[type]} model    [description]
       * @param  {[type]} schema   [description]
       * @return {[type]}          [description]
       */
      placeChanged(value, place, rawPlace, model, schema) {

        const addressComponents = rawPlace.address_components || []

        model.street = [
          this.extract(addressComponents, 'street_number'),
          this.extract(addressComponents, 'route', true)
        ].join(' ')

        model.city = this.extract(addressComponents, 'locality')
        model.state = this.extract(addressComponents, 'administrative_area_level_1', true)
        model.zipcode = this.extract(addressComponents, 'postal_code')
        model.country = this.extract(addressComponents, 'country')
      },


      /**
       * Posts data from the form on submit.
       * @param  {[type]} data [description]
       * @return {[type]}      [description]
       */
      submit (data) {

        // trigger loading spinner
        this.loading = true

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

              // reset error state
              this.error = null
            }

          })
          .catch((err) => {
            this.error = 'Sorry! It looks like there was an issue submitting the form. Please try again in a few minutes.'
          })
          .then(() => {
            // disable loading spinner
            this.loading = false
          })
      }
    }
  }
</script>

<style lang="scss">
  #jpjd-formwidget--app {
    display: flex;
    flex-direction: column;
    justify-content: center;
    min-height: 300px;

    .confirmation {
      margin: auto;
    }

    fieldset {
      border: 0;
    }
  }
</style>
