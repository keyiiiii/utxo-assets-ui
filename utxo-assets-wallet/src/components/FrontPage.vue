<template>
  <div>
    <div class="row break-word">
      Your public address: <h5 class="break-word">{{ address }}</h5>
    </div>
    <div class="row">
      Your balance: <h5>{{ balance }}</h5>
      Your genesis balance: <h5>{{ genesisBalance }}</h5>
    </div>
    <br>
    <h5>Send coins</h5>
    <form>
      <div class="row">
        <div class="twelve columns">
          <label for="receiverAddress">Receiver address</label>
          <input v-model="receiverAddress" class="u-full-width" type="text" placeholder="04f72a4541275aeb4344a8b04..." id="receiverAddress">
        </div>
        <div class="twelve columns">
          <label for="amount">Amount</label>
          <input v-model="receiverAmount" class="u-full-width" type="number" placeholder="0" id="amount">
        </div>
        <div class="twelve columns">
          <label for="assetId">AssetId</label>
          <select v-model="assetId" id="assetId">
            <option disabled value="">Please select one</option>
            <option>native</option>
            <option>genesis</option>
          </select>
        </div>
      </div>
      <button v-on:click="sendTransaction" class="button-primary">Send</button>
    </form>
    <h5>Transaction pool</h5>
    <div class="transaction" v-for="tx in transactionPool">
      <div class="row">
        <span >TxId: {{ tx.id }}</span>
      </div>
      <div class="row">
        <div class="five columns">
          <div v-for="txIn in tx.txIns">
            <div v-if="txIn.signature === ''">coinbase</div>
            <div class="break-word" v-else>{{ txIn.txHash }} {{ txIn.txOutIndex }}</div>
          </div>
        </div>
        <div class="one columns">
          ->
        </div>
        <div class="six columns">
          <div class="row" v-for="txOut in tx.txOuts">
            <div class="break-word">
              <span>address: {{ txOut.address }}</span>
              amount: {{ txOut.amount}} </div>
          </div>
        </div>
      </div>
    </div>
    <div v-if="transactionPool.length === 0"><span>No transactions in transaction pool</span></div>
    <br>
    <h5>Mine block</h5>
    <button v-on:click="mineBlock" class="button-primary">Click to mine block</button>
    <div>

    </div>
  </div>
</template>

<script>
  export default {
    name: 'FrontPage',
    data() {
      return {
        'address': null,
        'balance': null,
        'genesisBalance': null,
        'transactionPool': [],
        'receiverAddress': null,
        'receiverAmount' : null,
        'assetId': null,
      }
    },
    created() {
      this.init();
    },
    methods: {
      init: function() {
        this.getAddress();
        this.getBalance();
        this.getTransactionPool();
      },
      getAddress: function () {
        this.$http.get('/api/address')
          .then(resp => {
            this.address = resp.data.address;
          })
      },
      getBalance: function () {
        this.$http.get('/api/balance/native')
          .then(resp => {
            this.balance = resp.data.balance;
          });
        this.$http.get('/api/balance/genesis')
          .then(resp => {
            this.genesisBalance = resp.data.balance;
          })
      },
      sendTransaction: function() {
        this.$http.post('/api/sendTransaction',
          {'amount' : parseInt(this.receiverAmount), 'address' : this.receiverAddress, 'assetId': this.assetId}
          )
          .then(() => {
            this.receiverAmount = null;
            this.receiverAddress = null;
            this.assetId = null;
            this.init();
          })
      },
      mineBlock: function() {
        this.$http.post('/api/mineBlock')
          .then(() => {
            this.init();
        })
      },
      getTransactionPool: function() {
        this.$http.get('/api/transactionPool')
          .then((resp) => {
            this.transactionPool = resp.data;
          });
      }

    }
  }
</script>


<style scoped>
  .transaction {
    padding: 1em;
    margin-bottom: 1em;
    background-color: gainsboro;
  }
</style>
