<template>
  <q-page class="flex flex-center text-white bg-black">
    <q-card flat class="bg-black">
      <q-inner-loading :visible="spinnervisible">
        <q-spinner size="50px" color="primary" />
      </q-inner-loading>
      <q-card flat class="bg-black" style="width: 100%">
        <q-card-section class="text-weight-bold text-center">
          <q-icon class="float-left" name="help_outline" size="2.5rem" color="white" @click.native="$documentationManger.openDocumentation('exchange/coinswitch')">
            <q-tooltip>{{ $t('SettingsView.help') }}</q-tooltip>
          </q-icon>
          <big class="titillium text-uppercase">{{ $t('SettingsView.exchange') }}</big>
          <q-icon class="float-right" name="close" size="2.5rem" color="white" @click.native="$router.push('/wallet')"/>
        </q-card-section>
      </q-card>
      <q-card flat class="bg-black" style="width: 100%">
        <q-stepper v-model="step" done-color="green" active-color="green" ref="stepper" header-nav alternative-labels animated>
          <q-step
            default
            :name="1"
            title="Select Coin to Send"
            class="bg-black workflow-step"
            :done="step > 1"
            :header-nav="step > 1"
          >
            <q-card-section>
              <div class="text-center text-white uppercase">
                <div class="q-pa-md">
                  <q-select
                      dark
                      label="Select Coin to Send"
                      separator
                      v-model="depositCoin"
                      use-input
                      @filter="filterDepositCoin"
                      :disabled="!depositCoinOptions"
                      :loading="!depositCoinOptions"
                      :options="depositCoinOptions"
                    >
                    <template v-slot:option="scope">
                      <q-item
                        class="custom-menu"
                        v-bind="scope.itemProps"
                        v-on="scope.itemEvents"
                      >
                        <q-item-section avatar>
                          <q-icon :name="`img:${scope.opt.image}`" />
                        </q-item-section>
                        <q-item-section dark>
                          <q-item-label v-html="scope.opt.label" />
                          <q-item-label caption>{{ scope.opt.value }}</q-item-label>
                        </q-item-section>
                      </q-item>
                    </template>
                    <template v-slot:selected>
                      <q-item
                        v-if="depositCoin"
                      >
                        <q-item-section avatar>
                          <q-icon :name="`img:${depositCoin.image}`" />
                        </q-item-section>
                        <q-item-section>
                          <q-item-label v-html="depositCoin.label" />
                          <q-item-label caption>{{ depositCoin.value }}</q-item-label>
                        </q-item-section>
                      </q-item>
                      <q-item
                        v-else>
                      </q-item>
                    </template>
                  </q-select>
                  <q-input
                    type="text"
                    dark
                    v-model="refundAddress.address"
                    color="green"
                    @input="verifyAddress()"
                    :label="returnAddressLabel"
                  />
                  <q-input
                    type="text"
                    dark
                    v-model="refundAddress.tag"
                    color="green"
                    label="Optional tag or memo [some exchanges require this field]"
                  />
                </div>
                <div class="q-pa-sm" v-show="true" @click="checkToGetPairs()">
                  <q-icon name="navigate_next" size="3.2rem" color="green"   >
                    <q-tooltip>{{ $t('next') }}</q-tooltip>
                  </q-icon>
                </div>
              </div>
            </q-card-section>
          </q-step>
          <q-step
            :name="2"
            title="Select Destination Coin"
            class="bg-black workflow-step"
            :done="step > 2"
            :header-nav="step > 2"
          >
            <q-card-section>
              <div class="text-center text-white uppercase">
                <div class="q-pa-md">
                  <q-select
                      dark
                      label="Select Coin to Receive"
                      separator
                      use-input
                      @filter="filterDestinationCoin"
                      v-model="destinationCoin"
                      :disabled="!destinationCoinOptions"
                      :loading="!destinationCoinOptions"
                      :options="destinationCoinOptions"
                    >
                    <template v-slot:option="scope">
                      <q-item
                        class="custom-menu"
                        v-bind="scope.itemProps"
                        v-on="scope.itemEvents"
                      >
                        <q-item-section avatar>
                          <q-icon :name="`img:${scope.opt.image}`" />
                        </q-item-section>
                        <q-item-section>
                          <q-item-label v-html="scope.opt.label" />
                          <q-item-label caption>{{ scope.opt.value }}</q-item-label>
                        </q-item-section>
                      </q-item>
                    </template>
                    <template v-slot:selected>
                      <q-item
                        v-if="destinationCoin"
                      >
                        <q-item-section avatar>
                          <q-icon :name="`img:${destinationCoin.image}`" />
                        </q-item-section>
                        <q-item-section>
                          <q-item-label v-html="destinationCoin.label" />
                          <q-item-label caption>{{ destinationCoin.value }}</q-item-label>
                        </q-item-section>
                      </q-item>
                      <q-item
                        v-else>
                      </q-item>
                    </template>
                  </q-select>
                  <q-input
                    type="text"
                    dark
                    ref="destinationAddressAddress"
                    v-model="destinationAddress.address"
                    color="green"
                    @input="verifyAddress()"
                    :rules="[ val => val.length >= 3 || 'Destination Address Cannot less than 3 characters' ]"
                    :label="destinationAddressLabel"
                  />
                  <q-input
                    type="text"
                    dark
                    v-model="destinationAddress.tag"
                    color="green"
                    label="Optional tag or memo [some exchanges require this field]"
                  />
                </div>
                <div class="q-pa-sm" v-show="true" @click="checkToGetRate()">
                  <q-icon name="navigate_next" size="3.2rem" color="green"   >
                    <q-tooltip>{{ $t('next') }}</q-tooltip>
                  </q-icon>
                </div>
              </div>
            </q-card-section>
          </q-step>
          <q-step
            :name="3"
            title="Select Quantity"
            class="bg-black workflow-step"
            :done="step > 3"
            :header-nav="step > 3"
          >
            <q-card-section>
              <div class="text-center text-white uppercase">
                <q-item>
                  <q-item-label>Choose quantity</q-item-label>
                </q-item>
                <div class="q-pa-md">
                  <q-input
                    ref="depositQuantity"
                    class="q-pa-sm"
                    type="number"
                    dark
                    v-model="depositQuantity"
                    color="green"
                    @input="quantityFromDeposit()"
                    :disabled="!rateData"
                    :loading="!rateData"
                    :label="depositQuantityLabel"
                    :rules="[ val => val >= rateData.limitMinDepositCoin || 'This is less than the minimum allowed',
                              val => val < rateData.limitMaxDepositCoin || 'This is more than the maximum allowed']"
                  />
                  <q-input
                    ref="destinationQuantity"
                    class="q-pa-sm"
                    type="number"
                    dark
                    v-model="destinationQuantity"
                    color="green"
                    @input="quantityFromDestination()"
                    :disabled="!rateData"
                    :loading="!rateData"
                    :label="destinationQuantityLabel"
                    :rules="[ val => val >= rateData.limitMinDestinationCoin || 'This is less than the minimum allowed',
                              val => val < rateData.limitMaxDestinationCoin || 'This is more than the maximum allowed']"
                  />
                </div>
                <div class="q-pa-sm" v-show="true" @click="checkToPostOrder()">
                  <q-icon name="navigate_next" size="3.2rem" color="green"   >
                    <q-tooltip>{{ $t('next') }}</q-tooltip>
                  </q-icon>
                </div>
              </div>
            </q-card-section>
          </q-step>
          <q-step
            :name="4"
            title="View Order"
            class="bg-black workflow-step"
            :done="step > 4"
            :header-nav="step > 4"
          >
            <q-card-section>
              <div class="text-center text-white uppercase">
                <q-item>
                  <q-item-label>{{exchangeLabel}}</q-item-label>
                </q-item>
                <div class="col-auto flex flex-center">
                  <div class="q-pr-md">
                    <qrcode v-if="exchangeAddress.address" :value="exchangeAddress.address" :options="{size: 150}"></qrcode>
                  </div>
                </div>
                <q-btn class="full-width" no-caps flat @click="copy2clip(exchangeAddress.address)" size="sm">
                  <q-input class="fit"
                    dark
                    readonly
                    v-model="exchangeAddress.address"
                    hint=""
                  >
                    <template v-slot:append>
                      <q-icon name="file_copy" @click="copy2clip(exchangeAddress.address)" />
                    </template>
                  </q-input>
                </q-btn>
                <q-card dark bordered class="bg-grey-9 my-card">
                  <q-card-section>
                    <div class="text-h6">Status of the exchange checked every 30 seconds</div>
                  </q-card-section>
                  <q-separator dark inset />
                  <q-card-section>
                    <div class="row">
                      <div class="col">
                        {{ friendlyStatus }}
                      </div>
                      <div class="col">
                        <q-circular-progress
                          :indeterminate="!status"
                          show-value
                          :value="getStatus"
                          size="80px"
                          :thickness="0.25"
                          color="green"
                          :track-color="trackColor"
                        >
                          <q-avatar size="60px">
                            <img :src="`${logoUrl}`">
                          </q-avatar>
                        </q-circular-progress>
                      </div>
                    </div>
                  </q-card-section>
                </q-card>
              </div>
            </q-card-section>
          </q-step>
        </q-stepper>
      </q-card>
      <p class="text-caption text-center">
        * Disclaimer:  This transaction is carried out using an open API linked to various Exchanges.<br>
        By sending coins to the above address, you agreed to <a href="https://coinswitch.co/terms">the terms and conditions</a> of the selected provider.
      </p>
    </q-card>
  </q-page>
</template>

<script>
import store from '@/store'
import { userError } from '@/util/errorHandler'

const url = 'https://api.coinswitch.co'
let headers = {
  'x-api-key': process.env[store.state.settings.network].COINSWITCH_APIKEY
}

const typeUpper = function (thing) {
  if (typeof thing === 'string' && thing.length >= 1) {
    return thing.toUpperCase()
  } else {
    return ''
  }
}

export default {
  components: {
  },
  data () {
    return {
      step: 1,
      optionsSanitize: false,
      spinnervisible: false,
      lastChangedValue: 'deposit',
      coins: [],
      depositCoin: null,
      depositQuantity: 0,
      depositCoinOptions: null,
      depositCoinUnfilter: null,
      destinationCoin: null,
      destinationQuantity: 0,
      destinationCoinOptions: null,
      destinationCoinUnfilter: null,
      rateData: null,
      rateDataTemplate: {
        rate: 1,
        minerFee: 0,
        limitMinDepositCoin: 0,
        limitMaxDepositCoin: 1,
        limitMinDestinationCoin: 1,
        limitMaxDestinationCoin: 2
      },
      destinationAddress: {
        address: '',
        tag: null
      },
      refundAddress: {
        address: '',
        tag: null
      },
      exchangeAddress: {
        address: '',
        tag: null
      },
      expectedDepositCoinAmount: 0,
      expectedDestinationCoinAmount: 0,
      orderId: null,
      status: null,
      requestStop: false
    }
  },
  computed: {
    getStatus () {
      let value = 0

      switch (this.status) {
        case null:
        case 'no_deposit':
        case 'failed':
        case 'refunded':
        case 'timeout':
          value = 0
          break
        case 'confirming':
          value = 25
          break
        case 'exchanging':
          value = 50
          break
        case 'sending':
          value = 75
          break
        case 'complete':
          value = 100
          break
      }

      return value
    },
    friendlyStatus () {
      let value = ''

      switch (this.status) {
        case null:
          value = ''
          break
        case 'no_deposit':
          value = 'No deposit detected yet'
          break
        case 'failed':
          value = 'The transaction has failed'
          break
        case 'refunded':
          value = 'The transaction has been refunded'
          break
        case 'timeout':
          value = 'The transaction has timed out'
          break
        case 'confirming':
          value = 'The transaction is confirming'
          break
        case 'exchanging':
          value = 'The transaction is exchanging'
          break
        case 'sending':
          value = 'The coins are being sent'
          break
        case 'complete':
          value = 'The transaction is complete'
          break
      }

      return value
    },
    trackColor () {
      let value = ''

      switch (this.status) {
        case null:
        case 'no_deposit':
        case 'confirming':
        case 'exchanging':
        case 'sending':
        case 'complete':
          value = 'white'
          break
        case 'failed':
        case 'refunded':
        case 'timeout':
          value = 'red'
          break
      }

      return value
    },
    logoUrl () {
      if (this.destinationCoin != null) {
        return this.coins.filter(coins => coins.symbol === this.destinationCoin.value)[0].logoUrl
      } else {
        return '/static/icon.png'
      }
    },
    exchangeLabel () {
      if (this.depositCoin != null) {
        return 'Complete this exchange by sending ' + this.expectedDepositCoinAmount + ' ' + typeUpper(this.depositCoin.value) + ' to this address within the next 12 hours'
      } else {
        return 'Complete this exchange by sending the coins to this address within the next 12 hours'
      }
    },
    depositQuantityLabel () {
      if (this.depositCoin != null) {
        return typeUpper(this.depositCoin.value) + ' to Send'
      } else {
        return 'Coin to Send'
      }
    },
    destinationQuantityLabel () {
      if (this.destinationCoin != null) {
        return typeUpper(this.destinationCoin.value) + ' to Receive'
      } else {
        return 'Coin to Receive'
      }
    },
    returnAddressLabel () {
      if (this.depositCoin != null) {
        return 'Your ' + typeUpper(this.depositCoin.value) + ' return address [in case the transaction does not complete]'
      } else {
        return 'Your return address [in case the transaction does not complete]'
      }
    },
    destinationAddressLabel () {
      if (this.destinationCoin != null) {
        return 'Address to receive new ' + typeUpper(this.destinationCoin.value)
      } else {
        return 'Address to receive new coin'
      }
    }
  },
  created () {
  },
  mounted () {
    const self = this
    this.$axios.get(url + '/v2/coins', { headers }).then(function (result) {
      // will be using this coins array later with the destination select
      self.coins = result.data.data
      self.depositCoinOptions = self.coins.map(function (coin) {
        if (coin.isActive === true) {
          let row = {
            'label': coin.name,
            'value': coin.symbol,
            'image': coin.logoUrl
          }
          return row
        }
      }).filter(function (el) {
        return el != null
      }).sort(function (a, b) {
        if (a.label.toLowerCase() < b.label.toLowerCase()) {
          return -1
        }
        return 1
      })

      self.depositCoinUnfilter = self.depositCoinOptions
      console.log('depositCoinOptions', self.depositCoinOptions)
    })
  },
  methods: {
    filterDepositCoin (val, update, abort) {
      update(() => {
        const needle = val.toLowerCase()
        this.depositCoinOptions = this.depositCoinUnfilter.filter(v => v.value.toLowerCase().indexOf(needle) > -1)
      })
    },
    filterDestinationCoin (val, update, abort) {
      update(() => {
        const needle = val.toLowerCase()
        this.destinationCoinOptions = this.destinationCoinUnfilter.filter(v => v.value.toLowerCase().indexOf(needle) > -1)
      })
    },
    copy2clip (value) {
      // more generic copy
      this.$clipboardWrite(value)
      this.$q.notify({
        message: this.$t('Main.copied'),
        color: 'positive'
      })
    },
    checkToPostOrder () {
      if (this.$refs.depositQuantity.hasError || this.$refs.destinationQuantity.hasError) {
        userError('There is a problem with the quantities')
      } else {
        this.postOrder()
        this.$refs.stepper.next()
      }
    },
    checkToGetPairs () {
      if (this.depositCoin === null) {
        userError('There is a problem with the coin selection')
      } else {
        this.getPairs()
        this.$refs.stepper.next()
      }
    },
    checkToGetRate () {
      if (this.$refs.destinationAddressAddress.hasError || this.destinationAddress.address === '' ||
      this.destinationCoin === null) {
        userError('There is a problem with the destination address or the coin is not selected')
      } else {
        this.getRate()
        this.$refs.stepper.next()
      }
    },
    verifyAddress () {
      // check validity of all keys
    },
    quantityFromDeposit () {
      // deal with precision
      this.destinationQuantity = (+this.depositQuantity * +this.rateData.rate) - +this.rateData.minerFee
      this.lastChangedValue = 'deposit'
    },
    quantityFromDestination () {
      // deal with precision
      this.depositQuantity = (+this.destinationQuantity + +this.rateData.minerFee) / +this.rateData.rate
      this.lastChangedValue = 'destination'
    },
    orderStatus () {
      const self = this
      this.$axios.get(url + '/v2/order/' + this.orderId, { headers }).then(function (result) {
        self.status = result.data.data.status

        if (self.status === 'no_deposit' ||
        self.status === 'confirming' ||
        self.status === 'exchanging' ||
        self.status === 'sending') {
          setTimeout(() => { self.orderStatus() }, 30000)
        }
      })
    },
    postOrder () {
      const self = this
      let depositCoinAmount = null
      let destinationCoinAmount = null

      if (self.lastChangedValue === 'deposit') {
        depositCoinAmount = self.depositQuantity
      } else {
        destinationCoinAmount = self.destinationQuantity
      }

      this.$axios.post(url + '/v2/order',
        {
          depositCoin: self.depositCoin.value,
          destinationCoin: self.destinationCoin.value,
          depositCoinAmount,
          destinationCoinAmount,
          destinationAddress: self.destinationAddress,
          refundAddress: self.refundAddress
        },
        { headers })
        .then((response) => {
          self.orderId = response.data.data.orderId
          self.exchangeAddress = response.data.data.exchangeAddress
          self.expectedDepositCoinAmount = response.data.data.expectedDepositCoinAmount
          self.expectedDestinationCoinAmount = response.data.data.expectedDestinationCoinAmount

          this.orderStatus()
        })
        .catch((err) => {
          userError('There was a problem posting the order', err)
        })
    },
    getPairs () {
      const self = this
      this.$axios.post(url + '/v2/pairs',
        {
          depositCoin: self.depositCoin.value
        },
        { headers })
        .then((response) => {
          self.destinationCoinOptions = response.data.data.map(function (coin) {
            if (coin.isActive === true) {
              let row = {
                'label': self.coins.filter(coins => coins.symbol === coin.destinationCoin)[0].name,
                'value': coin.destinationCoin,
                'image': self.coins.filter(coins => coins.symbol === coin.destinationCoin)[0].logoUrl
              }
              return row
            } // deal with false, should not create empty option.
          }).filter(function (el) {
            return el != null
          }).sort(function (a, b) {
            if (a.label.toLowerCase() < b.label.toLowerCase()) {
              return -1
            }
            return 1
          })

          self.destinationCoinUnfilter = self.destinationCoinOptions
        })
        .catch((err) => {
          userError('There was a problem getting the destination coins', err)
        })
    },
    getRate () {
      const self = this
      this.$axios.post(url + '/v2/rate',
        {
          depositCoin: self.depositCoin.value,
          destinationCoin: self.destinationCoin.value
        },
        { headers })
        .then((response) => {
          self.rateData = response.data.data
        })
        .catch((err) => {
          userError('There was a problem getting the rate data', err)
        })
    }
  }
}
</script>

<style lang="stylus" scoped>
.custom-menu {
  background: #424242
}
.q-item__label {
  color: #ffffff;
}
.q-item__label--caption {
  color: #848484;
}
</style>
