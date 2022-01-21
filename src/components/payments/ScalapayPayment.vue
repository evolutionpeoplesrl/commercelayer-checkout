<template>
  <div class="payment-method">
    <v-radio
      :label="inputLabel('scalapay')"
      :value="payment_option.component"
      color="primary"
      @change="setPaymentMethod"
      id="paypal-payments-radio"
    ></v-radio>
    <div class="payment-method-fields" v-show="selected">
      <div id="scalapay-payment-hint">{{ $t('payment_methods.scalapay.hint') | capitalize }}</div>
      <div class="payment-error" id="scalapay-payment-error"></div>
    </div>
  </div>
</template>

<script>
import { paymentMixin } from '@/mixins/paymentMixin'

export default {
  mixins: [paymentMixin],
  methods: {
    paymentSourceAttributes () {
      return {
        payment_source_token: '0123456789'
      }
    },
    setupPayment () {
      let btn = document.getElementById('payment-step-submit')
      btn.onclick = () => {
        this.handlePayment()
      }
    },
    handlePaymentSourceError (error) {
      let scalapayError = document.getElementById('scalapay-payment-error')
      console.log(error)
      scalapayError.innerHTML = error.data.errors[0].detail
      this.loading_payment = false
    },
    handlePayment () {
      this.loading_payment = true
      console.log(this.order)

      const myHeaders = new Headers()
      myHeaders.append('Accept', 'application/json')
      myHeaders.append('Content-Type', 'application/json')

      const raw = JSON.stringify(this.order)

      const requestOptions = {
        method: 'POST',
        headers: myHeaders,
        body: raw,
        redirect: 'follow'
      }

      fetch('https://scalapay.fas-rocca.agenziadigital.it/api/v1/order', requestOptions)
        .then(res => res.json())
        .then(json => {
          console.log(json, json.hasOwnProperty('error'))
          if (json.hasOwnProperty('error')) {
            let errorObj = {}
            errorObj.data = {}
            errorObj.data.errors = [
              {
                'detail': json.error.message
              }
            ]
            console.log(errorObj)
            this.handlePaymentSourceError(errorObj)
          }
        })
        .catch(error => console.log('error', error))
    }
  }
}
</script>

<style>
</style>
