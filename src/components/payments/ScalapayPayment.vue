<template>
  <div class="payment-method">
    <v-radio
      :value="payment_option.component"
      color="primary"
      @change="setPaymentMethod"
      id="paypal-payments-radio"
    >
      <template v-slot:label>
        <span class="scalapay-method"><img class="scalapay-logo" src="/scalapay-logo.png" alt="Scalapay"><span class="scalapay-desc">Paga in 3 rate</span></span>
      </template>
    </v-radio>
    <div class="payment-method-fields" v-show="selected">
      <div id="scalapay-payment-hint" v-html="$t('payment_methods.scalapay.hint')"></div>
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
      scalapayInit(this.order).then(resp => {
        if (resp.hasOwnProperty('error')) {
          let errorObj = {}
          errorObj.data = {}
          errorObj.data.errors = [
            {
              'detail': resp.error
            }
          ]
          this.handlePaymentSourceError(errorObj)
          return
        }

        console.log('scalapay order created, redirecting to checkout url in 2s...')
        setTimeout(() => window.location.replace(resp.checkoutUrl), 2000)
      })
    }
  }
}
</script>

<style lang="scss">
.scalapay-method {
  display: flex;
  align-items: center;
}
.scalapay-logo {
  height: 22px;
}
.scalapay-desc {
  margin-left: 1rem;
  text-transform: uppercase;
  font-weight: bold;
  font-size: 11px;
  background: #4b3401;
  color: white;
  padding: 0 10px;
  border-radius: 3px;
}
#scalapay-payment-hint {
  font-size: 14px;

  a {
    text-decoration: underline;
  }
}
</style>
