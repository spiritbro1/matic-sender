<!-- Please remove this file from your project -->
<template>
  <div>
    <div class="center" v-show="!hilang">
      <div class="nes-container with-title is-centered">
        <p class="title">Send matic easily using unstoppable domain</p>
        <div class="kebawah" v-show="error">
          <span class="nes-text is-error">{{ error }}</span>
        </div>
        <div class="kebawah">
          <span class="nes-text is-warning"
            >For now can only send matic on mumbai network</span
          >
        </div>
        <div class="nes-field kebawah">
          <label for="name_field">Unstoppable Domain Name</label>
          <input
            type="text"
            id="name_field"
            v-model="calki"
            class="nes-input"
            @blur="poci"
          />
        </div>
        <div class="kebawah nes-text is-success">
          <a
            target="_blank"
            :href="`https://mumbai.polygonscan.com/address/${ud}`"
            >{{ ud }}</a
          >
        </div>
        <div class="kebawah nes-text is-error">{{ weror }}</div>
        <div class="nes-field kebawah">
          <label for="name_field">How much matic you want to send?</label>
          <input
            v-model="jumlah"
            type="number"
            id="name_field"
            class="nes-input"
          />
        </div>

        <button
          @click="cabi"
          :disabled="ud === '' && jumlah === 0 && typeof jumlah === 'number'"
          type="button"
          :class="{
            'nes-btn': true,
            'is-primary': true,
            'is-disabled':
              ud === '' || jumlah === 0 || typeof jumlah !== 'number',
          }"
        >
          Send now
        </button>
        <div class="keatas" v-if="sukses">
          <span class="nes-text is-success"
            >Success, see the transaction
            <a
              :href="`https://mumbai.polygonscan.com/tx/${transaction}`"
              target="_blank"
              >here</a
            ></span
          >
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import Web3 from 'web3-utils'

const hilang = ref(false);
const error = ref("");
const jumlah = ref(0);
const calki = ref("");
const weror = ref("");
const ud = ref("");
const sukses = ref(false);
const transaction = ref("");
const poci = async () => {
  weror.value = "";
  ud.value = "";
  if (calki.value !== "") {
    try {
      const res = await fetch(`/api/cakim?name=${calki.value}`);
      const hasil = await res.json();
      if (hasil.records["crypto.MATIC.version.MATIC.address"]) {
        ud.value = hasil.records["crypto.MATIC.version.MATIC.address"];
      } else {
        ud.value = "";
        weror.value = "we can't find any polygon address from that domain name";
      }
    } catch (e) {
      console.log(e.message);
      weror.value = "we can't find any polygon address from that domain name";
    }
  }
};
const cabi = async function () {
  if (!window.ethereum) return (error.value = "please install metamask");
  switchNetwork();
  await window.ethereum.enable();
  const accounts = await window.ethereum.request({
    method: "eth_requestAccounts",
  });
  const account = accounts[0];
  const value=Web3.toHex(Web3.toWei(jumlah.value.toString(), 'ether'));
  // const value = "";
  const transactionParameters = {
    to: ud.value,
    from: account, // must match user's active address.
    value, // Only required to send ether to the recipient from the initiating external account.
  };
  transaction.value = "";
  sukses.value = false;
  // txHash is a hex string
  // As with any RPC call, it may throw an error
  const txHash = await ethereum.request({
    method: "eth_sendTransaction",
    params: [transactionParameters],
  });
  if (txHash) {
    transaction.value = txHash;
    sukses.value = true;
  }
};
function switchNetwork() {
  let networkData;

  networkData = [
    {
      chainId: "0x13881",

      chainName: "polygon testnet",

      rpcUrls: ["https://matic-mumbai.chainstacklabs.com"],

      nativeCurrency: {
        name: "Polygon",

        symbol: "MATIC",

        decimals: 18,
      },

      blockExplorerUrls: ["https://mumbai.polygonscan.com/"],
    },
  ];

  // agregar red o cambiar red

  return window.ethereum.request({
    method: "wallet_addEthereumChain",

    params: networkData,
  });
}
</script>
