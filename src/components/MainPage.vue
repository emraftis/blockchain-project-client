<template>
  <h1>Main Page</h1>
  <div class="instructions" v-if="transactions.length < 1">
    <h2>Instructions:</h2>

  </div>


  <div class="search-box">
    <div class="search-input">
      <label for="yearInput">Year: </label>
      <select name="year" id="yearInput" v-model="year">
        <option value="2019">2019</option>
        <option value="2020">2020</option>
        <option value="2021">2021 (YTD)</option>
      </select>
    </div>
    
    <div class="search-input">
      <label for="reportInput">Report Type: </label>
      <select name="report" id="reportInput" v-model="report">
        <option value="txlist">ETH Transactions (Default)</option>
        <option value="tokentx">ERC20 Token Transactions</option>
        <option value="tokennfttx">ERC721 NFT Transactions</option>
      </select>
    </div>

    <div class="search-input">
      <label for="walletInput">Wallet Address: </label>
      <input type="text" id="walletInput" v-model.trim="wallet">
    </div>
    
    <button class="search-button" @click="fetchWalletTxns">Search</button>
  </div>


  <div class="loading" v-if="isLoading">
    <base-spinner></base-spinner>
  </div>
  <div class="list-container" v-if="!isLoading">
    <ul>
      <li v-for="transaction in transactions" :key="transaction">
        <div class="txn-card">
          <h2>{{ transaction.hash }}</h2>
          <ul>
            <li>Transaction Nonce: #{{ transaction.nonce }}</li>
            <li>From: {{ transaction.from }}</li>
            <li>To: {{ transaction.to }}</li>
            <li v-if="transaction.tokenSymbol">Token: {{ transaction.tokenName }} ({{ transaction.tokenSymbol }})</li>
            <li v-if="transaction.tokenSymbol">Qty: {{ transaction.value / (10 ** transaction.tokenDecimal) }}</li>
            <li v-if="!transaction.tokenSymbol">Transaction value (ETH): {{ convertGweitoETH(transaction.value) }}</li>
            <li v-if="!transaction.tokenSymbol">Transaction value (USD): {{ fromETHtoUSD(convertGweitoETH(transaction.value), transaction.data.amount) }}</li>
            <li>Date of Transaction: {{ getFullDate(transaction.formattedDate) }}</li>
            <li>Gas in ETH: {{ getGasETH(transaction.gasUsed, transaction.gasPrice) }}</li>
            <li>Gas in USD: {{ fromETHtoUSD(getGasETH(transaction.gasUsed, transaction.gasPrice), transaction.data.amount) }}</li>
          </ul>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
var formatter = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD'
})
import BaseSpinner from "./ui/BaseSpinner.vue";
export default {
  name: 'HelloWorld',
  components: {
    BaseSpinner,
  },
  data() {
    return {
      wallet: '', //0xAABF31b7dab0f236c4566054Cbc3673f43fc10F5
      year: '',
      report: '',
      isLoading: false,
      transactions: [],
    }
  },
  computed: {
    apiString() {
      return `/${this.year}/${this.wallet}?action=${this.report}`;
    },
    tokenSymbolQuantity(value, tokenDecimal) {
      console.log(value);
      console.log(tokenDecimal);
      return value;
    },
  },
  methods: {
    async fetchWalletTxns() {
      this.isLoading = true;
      try {
        const res = await fetch(`http://localhost:3000${this.apiString}`);
        this.transactions = await res.json();
        this.isLoading = false;
        return;
      } catch (e) {
        console.log(e);
        this.isLoading = false;
        return;
      } 
    },
    getFullDate(date) {
      const array = date.split("T")
      return array[0]
    },
    getGasETH(gasUsed, gasPrice) {
      return (gasUsed/1000000000) * (gasPrice/1000000000);      
    },
    fromETHtoUSD(eth, spotRate) {
      return formatter.format(eth * spotRate);
    },
    convertGweitoETH(gwei) {
      return gwei/1000000000000000000;
    },
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h2 {
  font-size: 1rem;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  margin: 0.25rem;
}
a {
  color: #42b983;
}
.txn-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 100%;
  font-size: 1rem;
  border: 1px solid black;
  margin: 2rem 15%;
  padding: 1rem 0;
}
.search-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
}
.loading {
  margin-top: 5rem;
}
.search-input {
  display: flex;
  padding: 0.25rem 0;
}
.search-input label {
  padding: 0 0.5rem;
}
.search-button {
  margin: 0.5rem;
}
</style>
