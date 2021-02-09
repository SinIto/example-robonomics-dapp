<template>
  <div id="app">
    <h1>Robonomics dApp</h1>
    <div v-if="load">...</div>
    <template v-else>
      <div v-if="error" class="error">{{ error }}</div>
      <template v-else-if="api && account">
        <p>
          Account: <b>{{ account }}</b> {{ balance }} |
          <button @click="faucet">faucet</button>
        </p>

        <div>
          <div class="tabs">
            <button
              @click="tab = 'datalog'"
              :class="{ active: tab === 'datalog' }"
            >
              datalog
            </button>
            <button
              @click="tab = 'launch'"
              :class="{ active: tab === 'launch' }"
            >
              launch
            </button>
          </div>
          <Datalog v-if="tab === 'datalog'" :api="api" :account="account" />
          <Launch v-if="tab === 'launch'" :api="api" :account="account" />
        </div>
      </template>
    </template>
  </div>
</template>

<script>
import Datalog from "./components/Datalog";
import Launch from "./components/Launch";
import { initApi, initAccount, getBalance, faucet } from "./utils/api";
import { formatBalance } from "@polkadot/util";
export default {
  name: "App",
  components: {
    Datalog,
    Launch
  },
  data() {
    return {
      load: false,
      api: null,
      account: null,
      balance: 0,
      error: null,
      tab: "datalog"
    };
  },
  created() {
    this.init();
  },
  methods: {
    async init() {
      try {
        this.load = true;
        this.api = await initApi();
        this.account = await initAccount();
        getBalance(this.account, balance => {
          this.balance = formatBalance(balance);
        });
        this.load = false;
      } catch (error) {
        this.error = error.message;
        this.load = false;
      }
    },
    faucet() {
      faucet(this.account);
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
button {
  font-size: 14px;
  padding: 5px 12px;
}
button:hover {
  cursor: pointer;
}
input {
  font-size: 14px;
  padding: 5px;
}
select {
  font-size: 14px;
  padding: 5px;
}
button:focus,
input:focus {
  outline: none;
}
.error {
  color: rgb(151, 31, 31);
  font-weight: bold;
  text-align: center;
  margin: 10px 0;
}
.tabs button {
  font-size: 14px;
  padding: 10px 20px;
  font-weight: bold;
  background: #ececec;
  border: 1px solid #aaa;
}
.tabs button:hover {
  background: #bfbfbf;
}
.tabs button:last-child {
  border-left: none;
}
.tabs button.active {
  background: #ced5e2;
}
</style>
