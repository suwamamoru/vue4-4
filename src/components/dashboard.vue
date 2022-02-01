<template>
  <div class="dashboard">
    <img alt="Vue logo" src="../assets/logo.png">
    <div class="login-info">
      <span class="login-user">{{$store.getters.signinUser}}さんようこそ!!</span>
      <span class="wallet">残高:{{$store.getters.signinWallet}}</span>
      <span><button class="logout-button" @click="logout()">ログアウト</button></span>
    </div>
    <div class="users">
      <h1>ユーザ一覧</h1>
      <table class="users">
        <tr>
          <td class="user-name">ユーザー名</td>
          <td><br></td>
          <td><br></td>
        </tr>
        <tr v-for="(userData,index) in $store.getters.receiveUsers" :key="index">
          <td class="user-name">{{userData}}</td>
          <td><button class="wallet-buttons" @click="openWallet(index)">walletを見る</button></td>
          <td><button class="wallet-buttons" @click="openSubmit(index)">送る</button></td>
        </tr>
      </table>
    </div>
    <modal @close="closeWallet()" v-if="modalStatus.walletModal">
      <p>{{watch.watchUser}}さんの残高</p>
      <p>{{watch.watchWallet}}</p>
      <template slot="footer">
        <button class="close-buttons" @click="closeWallet()">close</button>
      </template>
    </modal>
    <modal @close="closeSubmit()" v-if="modalStatus.submitModal">
      <p>あなたの残高:{{$store.getters.signinWallet}}</p>
      <p>送る金額</p>
      <input type="number" min="0" v-model.number="cal.giveWallet">
      <p class="error" v-show="error.errorShow">{{error.errorMessage}}</p>
      <template slot="footer">
        <button class="close-buttons" @click="closeSubmit()">送信</button>
      </template>
    </modal>
    <div class="copyright">
      <p>Copyright ©️2021 oo Inc. All rights reserved.</p>
    </div>
  </div>
</template>

<script>
import modal from './modal.vue'

export default {
  components: {
    modal
  },
  data() {
    return {
      modalStatus: {
        walletModal: false,
        submitModal: false
      },
      watch: {
        watchId: '',
        watchUser: '',
        watchWallet: ''
      },
      error: {
        errorShow: false,
        errorMessage: '残高を超える投げ銭はできません。'
      },
      cal: {
        giveWallet: '',
        afterSigninWallet: '',
        totalWallet: ''
      }

    }
  },
  mounted() {
    this.startListener()
  },
  beforeDestroy() {
    this.stopListener()
  },
  computed: {
    computedWatchId: {
      get() {
        return this.$store.getters.updateWatchId
      },
      set(value) {
        this.$store.dispatch("getWatchId", value)
      }
    },
    computedAfterSigninWallet: {
      get() {
        return this.$store.getters.updateAfterSigninWallet
      },
      set(value) {
        this.$store.dispatch("getAfterSigninWallet", value)
      }
    },
    computedTotalWallet: {
      get() {
        return this.$store.getters.updateTotalWallet
      },
      set(value) {
        this.$store.dispatch("getTotalWallet", value)
      }
    },
  },
  methods: {
    startListener() {
      this.$store.dispatch('startListener')
    },
    stopListener() {
      this.$store.dispatch('stopListener')
    },
    logout() {
      this.$store.dispatch('logout')
    },
    openWallet(index) {
      this.watch.watchUser = this.$store.getters.receiveUsers[index]
      this.watch.watchWallet = this.$store.getters.receiveWallets[index]
      this.modalStatus.walletModal = true
    },
    closeWallet() {
      this.modalStatus.walletModal = false
    },
    openSubmit(index) {
      this.watch.watchId = this.$store.getters.receiveIds[index]
      this.watch.watchWallet = this.$store.getters.receiveWallets[index]
      this.cal.giveWallet = ''
      this.cal.totalWallet = ''
      this.error.errorShow = false
      this.modalStatus.submitModal = true
    },
    closeSubmit() {
      if(this.cal.giveWallet === '') {
        this.cal.giveWallet = 0
      }
      this.cal.afterSigninWallet = this.$store.getters.signinWallet - this.cal.giveWallet
      if(this.cal.afterSigninWallet < 0) {
        this.error.errorShow = true
        return
      }
      this.cal.totalWallet = this.watch.watchWallet + this.cal.giveWallet
      this.computedWatchId = this.watch.watchId
      this.computedAfterSigninWallet = this.cal.afterSigninWallet
      this.computedTotalWallet = this.cal.totalWallet
      this.$store.dispatch('submit')
      //this.$store.dispatch('getData')
      this.modalStatus.submitModal = false
    }
  }
}
</script>

<style>
.login-user {
  margin-right: 30px;
  text-align: left;
}

.wallet {
  margin-right: 30px;
  text-align: right;
}

.logout-button {
  background-color: #fff;
  border-radius: 0.5vh;
  border: 1px solid #6495ed;
  color: #6495ed;
}

.users {
  margin: auto;
}

.user-name {
  width: 200px;
}

.wallet-buttons {
  background-color: #20b2aa;
  border-radius: 0.5vh;
  border: 1px solid #20b2aa;
  color: #fff;
}

.error {
  font-size: 6px;
  color: red;
}

.close-buttons {
  background-color: #ff0000;
  border-radius: 0.5vh;
  border: 1px solid #cc0000;
  color: #fff;
}
</style>