<template>
  <h1>ETH Gas Wallet Tracker</h1>
  <div class="instructions">
    <h2>User Guide:</h2>
    <ul class="user-guide-list">
      <li>This application shows accumulated Gas Fees for a wallet address.</li>
      <li>Please enter the year and a <span class="strong-font">public, non-custodial</span> wallet address that you want to search.</li>
      <li>Transactions that show a value of '0 ETH' are usually ERC20 token transactions.</li>
      <li>You can find out more about these '0 ETH Transactions' by changing the Report Type.</li>
      <li>Click the bold transaction hash to see this transaction on Etherscan.</li>
      <li>Please keep in mind this is a free service and it will take 10-20 seconds to load each wallet.</li>
      <li>Want to test it? Try the <a class="underline-font" href="https://www.coindesk.com/markets/2021/08/10/cross-chain-defi-site-poly-network-hacked-hundreds-of-millions-potentially-lost/">Poly Network Hacker's ETH address...</a> 0xC8a65Fadf0e0dDAf421F28FEAb69Bf6E2E589963</li>
    </ul>
  </div>


  <div class="search-box">
    <div class="search-input">
      <label for="yearInput">Year: </label>
      <select name="year" id="yearInput" v-model="year" @change="valueChange">
        <option value="2019">2019</option>
        <option value="2020">2020</option>
        <option value="2021">2021 (YTD)</option>
      </select>
    </div>
    
    <div class="search-input">
      <label for="reportInput">Report Type: </label>
      <select name="report" id="reportInput" v-model="report" @change="valueChange">
        <option value="txlist">ETH Transactions (Default)</option>
        <option value="tokentx">ERC20 Token Transactions</option>
      </select>
    </div>

    <div class="search-input">
      <label for="walletInput">Wallet Address: </label>
      <input type="text" id="walletInput" v-model.trim="wallet" @change="valueChange">
    </div>
    
    <div class="button-block">
      <button class="search-button" @click="fetchWalletTxns">Search</button>
    </div>
    
  </div>


  <div class="loading" v-if="isLoading">
    <h2 v-if="year==='2019'">Searching 2,204,339 blocks...</h2>
    <p v-if="year==='2019'">This might take 30+ seconds!</p>
    <h2 v-if="year==='2020'">Searching 2,372,282 blocks...</h2>
    <p v-if="year==='2020'">This might take 30+ seconds!</p>
    <h2 v-if="year==='2021'">Searching 1,500,000+ blocks...</h2>
    <p v-if="year==='2021'">This might take 30+ seconds!</p>
    <base-spinner></base-spinner>
  </div>

  <div class="summary-container" v-if="transactions.length > 0 && !isLoading">
    <ul>
      <li v-if="report==='txlist'">Number of Transactions Sent in {{ year }}: <span class="strong-font">{{ transactions.length }}</span></li>
      <li v-if="report==='tokentx'">Number of ERC20 Transactions Sent in {{ year }}: {{ transactions.length }}</li>
    </ul>
  </div>

  <div class="list-container" v-if="!isLoading">
    <div v-if="noResults">
      <h2>No results!</h2>
    </div>
    
    <ul>
      <li v-for="transaction in transactions" :key="transaction">
        <p>Date: {{ getFullDate(transaction.formattedDate) }} | Nonce: #{{ transaction.nonce }}</p>
        <div class="txn-card">
          <div class="card-heading">
              <h2 class="card-header">Tx Hash: <a class="strong-font" :href="linkToEtherscanTx(transaction.hash)">{{ transaction.hash }}</a></h2>
          </div>

          <ul>
            <li>From: <a class="strong-font" :href="linkToEtherscanAc(transaction.from)">{{ transaction.from }}</a></li>
            <li>To: <a class="strong-font" :href="linkToEtherscanAc(transaction.to)">{{ transaction.to }}</a></li>
            <li></li>
            <li></li>
            <li v-if="transaction.tokenSymbol">Token(s): {{ transaction.value / (10 ** transaction.tokenDecimal) }} x {{ transaction.tokenName }} ({{ transaction.tokenSymbol }})</li>
            <li v-if="!transaction.tokenSymbol">Transaction value: <i class="fab fa-ethereum"></i> {{ convertGweitoETH(transaction.value) }} (USD{{ fromETHtoUSD(convertGweitoETH(transaction.value), transaction.data.amount) }})</li>
            <li class="strong-font">Gas paid: <i class="fab fa-ethereum"></i> {{ getGasETH(transaction.gasUsed, transaction.gasPrice) }} (USD{{ fromETHtoUSD(getGasETH(transaction.gasUsed, transaction.gasPrice), transaction.data.amount) }})</li>
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
  name: 'MainProject',
  components: {
    BaseSpinner,
  },
  data() {
    return {
      wallet: '',
      year: '2021',
      report: 'txlist',
      isLoading: false,
      transactions: [],
      noResults: false,
    }
  },
  computed: {
    apiString() {
      return `/${this.year}/${this.wallet}?action=${this.report}`;
    },
  },
  methods: {
    async fetchWalletTxns() {
      this.noResults = false;
      this.isLoading = true;
      try {
        const res = await fetch(`${this.apiString}`);
        this.transactions = await res.json();
        if (this.transactions.length < 1) {
          this.noResults = true;
        }
        this.isLoading = false;
        return;
      } catch (e) {
        console.log(e);
        this.isLoading = false;
        return;
      } 
    },
    valueChange() {
      this.transactions = [];
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
    linkToEtherscanTx(hash) {
      return "https://etherscan.io/tx/" + hash.toString();
    },
    linkToEtherscanAc(acc) {
      return "https://etherscan.io/address/" + acc.toString();
    }
  },
}
</script>


<style scoped>
h2 {
  font-size: 1.25rem;
}
p {
  margin: 0;
  padding: 0;
}
a {
  text-decoration: none;
  color: rgb(38, 67, 119);
}
.underline-font {
  text-decoration: underline;
}
.card-header {
  font-weight: 400;
  margin: 0;
  padding: 0;
}
ul {
  list-style-type: none;
  padding: 0;
  width: fit-content;
  font-size: 1.25rem;
}
.user-guide-list {
  margin-left: 2rem;
  padding-left: 1rem;
  list-style-type: style;
  font-size: 1.15rem;
  line-height: 1.25rem;
}
li {
  margin: 0;
  padding: 0;
  text-align: left;
}
select, option, label, input {
  font-size: 1.2rem;
}
.txn-card {
  display: relative;
  font-size: 1rem;
  border: 1px solid black;
  font-family: 'News Cycle', sans-serif;
  margin-bottom: 2rem;
}
.txn-card ul {
  padding: 1rem;
}
.txn-card ul li {
  padding: 0.15rem;
}
.strong-font {
  font-weight: 600;
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
.list-container {
  display: flex;
  justify-content: center;
}
.summary-container,
.instructions {
  display: flex;
  justify-content: center;
}
.instructions {
  padding: 2rem 0;
  border-top: 1px solid black;
  border-bottom: 1px solid black;
  margin-bottom: 1rem;
}
</style>
