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

const backendInitUrl = 'https://scalapay.fas-rocca.agenziadigital.it/api/v1/order/init'

async function scalapayInit (orderPayload) {
  const myHeaders = new Headers()
  myHeaders.append('Accept', 'application/json')
  myHeaders.append('Content-Type', 'application/json')

  const raw = JSON.stringify(orderPayload)

  const requestOptions = {
    method: 'POST',
    headers: myHeaders,
    body: raw,
    redirect: 'follow'
  }

  const response = await fetch(backendInitUrl, requestOptions)
  const result = await response.json()

  return result
}

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
      let scalapayError = document.getElementById('scalapay-payment-error')
      scalapayError.innerHTML = ''
    },
    handlePaymentSourceError (error) {
      let scalapayError = document.getElementById('scalapay-payment-error')
      let errorMessage = document.createElement('div')
      errorMessage.style.fontSize = '13px'
      errorMessage.style.color = 'red'
      errorMessage.innerText = this.$t('errors.processing_payment')
      scalapayError.append(errorMessage)
      let errorDetails = document.createElement('div')
      errorDetails.style.fontSize = '11px'
      errorDetails.style.fontFamily = 'monospace'
      errorDetails.innerHTML = error.data.errors[0].detail
      scalapayError.append(errorDetails)
      this.loading_payment = false
    },
    handlePayment () {
      this.loading_payment = true
      // this.$store.dispatch('placeOrder', this.order)
      scalapayInit().then(resp => {
        console.log(resp)
      })
      /* console.log(this.order)

      const fetchHeaders = new Headers()
      fetchHeaders.append('Accept', 'application/json')
      fetchHeaders.append('Content-Type', 'application/json')

      const raw = JSON.stringify(this.order)

      const requestOptions = {
        method: 'POST',
        headers: fetchHeaders,
        body: raw,
        redirect: 'follow'
      }

      fetch(scalapayBackendBaseUrl + '/order', requestOptions)
        .then(res => res.json())
        .then(json => {
          if (json.hasOwnProperty('error')) {
            let errorObj = {}
            errorObj.data = {}
            errorObj.data.errors = [
              {
                'detail': json.error.message
              }
            ]
            this.handlePaymentSourceError(errorObj)
            return
          }

          console.log(json)
        })
        .catch(error => console.log('error', error)) */
    }
  }
}
</script>

<style lang="scss">

</style>
