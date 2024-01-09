<template>
  <v-container class="fill-height">
    <v-responsive class="align-center text-center fill-height">
      <div class="text-h2 font-weight-light mb-n1">Issue with vite build</div>

      <div class="py-14" />
      <v-row class="d-flex align-center justify-center" v-if="!phrase || errors">
        <v-col cols="12">
          <v-textarea
            variant="filled"
            label="Enter your mnemonic phrase to connect"
            v-model="phrase"
            :class="{'v-field--error': errors }"
            :error-messages="errorPhrase"
          ></v-textarea>
        </v-col>
        </v-row>

      <div class="py-14" />

      <v-row class="d-flex align-center justify-center" v-if="thorAddress">
        <v-col cols="auto">
          address: {{ thorAddress }}
        </v-col>

        <v-col cols="auto">
          with balance: {{ thorBalance || 0}}
        </v-col>
      </v-row>
      <v-row class="d-flex align-center justify-center" v-if="kujiAddress">
        <v-col cols="auto">
          address: {{ kujiAddress }}
        </v-col>

        <v-col cols="auto">
          with balance: {{ kujiBalance || 0 }}
        </v-col>
      </v-row>
      <v-row class="d-flex align-center justify-center">
      <v-col cols="auto">
        <v-btn href="https://github.com/mihoz-fe/xchainjs-vue3-vite" min-width="164" rel="noopener noreferrer" target="_blank"
          variant="text">
          <v-icon icon="mdi-github" size="large" start />
          Repository
        </v-btn>
      </v-col>
      </v-row>
    </v-responsive>
  </v-container>
</template>

<script lang="ts" setup>
import { Client, } from '@xchainjs/xchain-thorchain'
import { baseToAsset } from "@xchainjs/xchain-util"
import { computed, ref, watch } from 'vue';
import { Client as KujiraClient } from "@xchainjs/xchain-kujira"

const thorAddress = ref('')
const thorBalance = ref<Number | string>('')
const kujiAddress = ref('')
const kujiBalance = ref<Number | string>('')
const errors = ref<string>('')
const errorPhrase = computed(()=>{
  return errors.value ? 'invalid phrase' : ''
})

const phrase = ref<string>(import.meta.env.VITE_PHRASE || '')


// Create new instance of the client and query chain for balances. 
const connectWallet = async () => {

  try {
    const thorClient = new Client({ phrase: phrase.value })
    const address = await thorClient.getAddressAsync()
    thorAddress.value = address
    const balance = await thorClient.getBalance(address)
    const assetAmount = (baseToAsset(balance[0].amount)).amount()
    console.log(`With balance: ${assetAmount}`)
    thorBalance.value = assetAmount.toNumber()
  } catch (error: any) {
    // console.log(`Caught ${error}`, error,  typeof error)
    errors.value = error as string
    console.log(typeof errors.value, errors.value)
  }

  const kujiClient = new KujiraClient({ phrase: phrase.value })
  const addressKuji = await kujiClient.getAddressAsync()
  kujiAddress.value = addressKuji
  try {
    const balance = await kujiClient.getBalance(addressKuji)
    const assetAmount = (baseToAsset(balance[0].amount)).amount()
    console.log(`With balance: ${assetAmount}`)
    kujiBalance.value = assetAmount.toNumber()
  } catch (error) {
    console.error(`kuji balance ${error}`)
  }
}
watch(phrase, () => {
  connectWallet()
})
if(phrase.value)connectWallet()
</script>
