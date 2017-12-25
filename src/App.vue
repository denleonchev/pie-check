<template>
  <div class="page-container">
    <md-app md-waterfall md-mode="fixed">
      <md-app-toolbar class="md-primary">
        <img class="dog" src="./assets/dogesmall.jpg" alt="dog">
        <span class="md-title">PIE-check</span>
      </md-app-toolbar>
      <md-app-content>
        <form novalidate @submit.prevent="validate">
          <div class="md-layout md-gutter">
            <div class="md-layout-item md-large-size-50 md-medium-size-100">
              <md-card class="md-elevation-5">
                <md-card-header>
                  <div class="md-title">Enter data from the reviews report here</div>
                </md-card-header>
                <md-card-content>
                  <md-field :class="getValidationClass('emailsToCheck')">
                    <label>Emails of the contributors, who actually wrote reviews</label>
                    <md-textarea v-model="form.emailsToCheck" required></md-textarea>
                    <span class="md-error" v-if="!$v.form.emailsToCheck.required">The field is required</span>
                  </md-field>
                  <md-field :class="getValidationClass('productIDsToCheck')">
                    <label>IDs of the products, for which reviews were actually written</label>
                    <md-textarea v-model="form.productIDsToCheck" required></md-textarea>
                    <span class="md-error" v-if="!$v.form.productIDsToCheck.required">The field is required</span>
                  </md-field>
                </md-card-content>
              </md-card>
            </div>
            <div class="md-layout-item md-large-size-50 md-medium-size-100">
              <md-card class="md-elevation-5">
                <md-card-header>
                  <div class="md-title">Enter data from the purchases report here</div>
                </md-card-header>
                <md-card-content>
                  <md-field :class="getValidationClass('emails')">
                    <label>Emails of the contributors, who could have written reviews</label>
                    <md-textarea v-model="form.emails" required></md-textarea>
                    <span class="md-error" v-if="!$v.form.emails.required">The field is required</span>
                  </md-field>
                  <md-field :class="getValidationClass('productIDs')">
                    <label>IDs of the products, for which reviews could have been written reviews</label>
                    <md-textarea v-model="form.productIDs" required></md-textarea>
                    <span class="md-error" v-if="!$v.form.productIDs.required">The field is required</span>
                  </md-field>
                </md-card-content>
              </md-card>
            </div>
          </div>
          <div class="md-layout md-gutter md-alignment-center-center buttons-container">
            <md-button type="submit" class="md-raised md-primary"><md-icon md-font-set="material-icons">compare_arrows</md-icon>Calculate</md-button>
            <md-button @click="clearForm" class="md-raised md-accent"><md-icon md-font-set="material-icons">delete</md-icon>Clear</md-button>
            <md-button class="md-raised md-primary" v-if="wasCalculated" v-clipboard:copy="calculated" ><md-icon md-font-set="material-icons">work</md-icon>Copy</md-button>
          </div>
          </form>
      <md-bottom-bar>Use of this form assumes that you have two reports: reviews report and purchases report. Copy the necessary cells from reports and insert them to the respective fields. The values should be separated by a new line. When the form is filled, click "Calculate" and "Copy". The output will be copied to the buffer and you will be able to insert it to the spreadsheet column
      </md-bottom-bar>
      </md-app-content>
    </md-app>
  </div>
</template>

<script>
import { validationMixin } from 'vuelidate'
import { required } from 'vuelidate/lib/validators'
export default {
  mixins: [validationMixin],
  data: () => ({
    form: {
      emailsToCheck: null,
      productIDsToCheck: null,
      emails: null,
      productIDs: null
    },
    calculated: '',
    wasCalculated: false
  }),
  validations: {
    form: {
      emailsToCheck:  {
        required
      },
      productIDsToCheck:  {
        required
      },
      emails: {
        required
      },
      productIDs: {
        required
      }
    }
  },
  computed: {
    arrEmailsToCheck: function () {
      return this.separate(this.form.emailsToCheck)
    },
    arrProductIDsToCheck: function () {
      return this.separate(this.form.productIDsToCheck)
    },
    arrEmails: function () {
      return this.separate(this.form.emails)
    },
    arrProductIDs: function () {
      return this.separate(this.form.productIDs)
    }
  },
  methods: {
    separate(string) {
      return string.split(/\n/)
    },
    calculate() {
      const mergedProducts = []
      const arrEmailsToCheck =  this.separate(this.form.emailsToCheck)
      const arrProductIDsToCheck = this.separate(this.form.productIDsToCheck)
      const arrEmails = this.separate(this.form.emails)
      const  arrProductIDs = this.separate(this.form.productIDs)

      arrEmailsToCheck.forEach((email, i) => {
        const foundEmailIndex = arrEmails.indexOf(email)
        foundEmailIndex > -1 ? mergedProducts.push(arrProductIDs[foundEmailIndex]) : mergedProducts.push(arrProductIDsToCheck[i])
      })
      this.calculated = mergedProducts.join("\n")
      this.wasCalculated = true
    },
    hideCopy() {
      this.wasCalculated = false
    },
    getValidationClass (fieldName) {
      const field = this.$v.form[fieldName]
      if (field) {
        return {
          'md-invalid': field.$invalid && field.$dirty
        }
      }
    },
    validate() {
      this.$v.$touch()
      if (!this.$v.$invalid) {
        this.calculate()
      }
    },
    clearForm () {
      this.$v.$reset()
      this.form.emailsToCheck = ''
      this.form.productIDsToCheck = ''
      this.form.emails = ''
      this.form.productIDs = ''
    }
  },
  watch: {
    'form.emailsToCheck': function () {
      this.wasCalculated = false
    },
    'form.productIDsToCheck': function () {
      this.wasCalculated = false
    },
    'form.emails': function () {
      this.wasCalculated = false
    },
    'form.productIDs': function () {
      this.wasCalculated = false
    }
  }
}
</script>

<style lang="scss" scoped>
@import "~vue-material/dist/theme/engine"; 
@include md-register-theme("default", (
  primary: md-get-palette-color(blue, A200)
));
@import "~vue-material/dist/theme/all";

.md-card {margin-bottom: 20px;}
.dog {width: 56px; border-radius: 50%; transition: transform 1s ease-out 0s;}
  .dog:hover {transform: rotate(360deg)}
button {line-height: 26px;}
.md-bottom-bar {margin-top: 50px;}
</style>
