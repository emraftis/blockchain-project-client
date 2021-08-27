<template>
  <h1>Main Page</h1>
  <div class="list-container">
    <ul>
      <li v-for="transaction in transactions" :key="transaction">
        <div class="txn-card">
          <h2>{{ transaction.hash }}</h2>
          <ul>
            <li>From: {{ transaction.from }} - Nonce #{{ transaction.nonce }}</li>
            <li>To: {{ transaction.to }}</li>
            <li>Transaction value (ETH): {{ convertGweitoETH(transaction.value) }}</li>
            <li>Transaction value (USD): {{ fromETHtoUSD(convertGweitoETH(transaction.value), transaction.data.amount) }}</li>
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
export default {
  name: 'HelloWorld',
  data() {
    return {
      wallet: '0xAABF31b7dab0f236c4566054Cbc3673f43fc10F5',
      isLoading: false,
      transactions: [],
    }
  },
  methods: {
    async fetchWalletTxns2021() {
      this.isLoading = true;
      try {
        const res = await fetch(`http://localhost:3000/2021/${this.wallet}?action=txlist`);
        this.transactions = await res.json();
        this.isLoading = false;
        return;
      } catch (e) {
        console.log(e)
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
    }
  },
  created() {
    this.fetchWalletTxns2021();
  }
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
</style>
