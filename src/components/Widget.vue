<template>
  <div class='panel-body'>
    <vue-form-generator :schema="schema" :model="model" :options="formOptions"></vue-form-generator>
  </div>
</template>

<script>
  import VueFormGenerator from 'vue-form-generator/dist/vfg-core.js'
  import 'vue-form-generator/dist/vfg-core.css'

  export default {
    components: {
      'vue-form-generator': VueFormGenerator.component
    },

    data: function () {
      return {

        model: {
          firstName: null,
          lastName: null,
          email: null
        },

        schema: {
          fields: [{
            type: 'input',
            inputType: 'text',
            label: 'First Name',
            model: 'firstName',
            required: true,
            validator: VueFormGenerator.validators.string
          }, {
            type: 'input',
            inputType: 'text',
            label: 'Last Name',
            model: 'lastName',
            required: true,
            validator: VueFormGenerator.validators.string
          }, {
            type: 'input',
            inputType: 'email',
            label: 'E-mail',
            model: 'email',
            placeholder: 'Email',
            validator: VueFormGenerator.validators.email
          }, {
            type: 'submit'
          }]
        },

        formOptions: {
          validateAfterLoad: false,
          validateAfterChanged: true
        }
      }
    }
}
</script>
