<template>
  <div id="app">
    <h2>Login With Web3Auth</h2>
    <Loader :isLoading="loading"></Loader>
    <section
      :style="{
        fontSize: '12px',
      }"
    ></section>
    <section
      :style="{
        fontSize: '12px',
      }"
    >
      <button class="rpcBtn" v-if="!provider" name="google" @click="connect" style="cursor: pointer">Login With Google</button>
      <button class="rpcBtn" v-if="!provider" name="facebook" @click="connect" style="cursor: pointer">Login With facebook</button>
      <button class="rpcBtn" v-if="provider" @click="logout" style="cursor: pointer">Logout</button>
      <button class="rpcBtn" v-if="provider" @click="getUserInfo" style="cursor: pointer">Get User Info</button>
      <EthRpc v-if="provider" :provider="provider" :console="console"></EthRpc>

      <!-- <button @click="showError" style="cursor: pointer">Show Error</button> -->
    </section>
    <div id="console" style="white-space: pre-line">
      <p style="white-space: pre-line"></p>
    </div>
  </div>
</template>

<script lang="ts">
import { ADAPTER_STATUS, CHAIN_NAMESPACES, CONNECTED_EVENT_DATA, WALLET_ADAPTERS } from "@web3auth/base";
import { Web3AuthCore } from "@web3auth/core";
import { OpenloginAdapter, OpenloginLoginParams } from "@web3auth/openlogin-adapter";
import Vue from "vue";

import Loader from "../components/loader.vue";
import config from "../config";
import EthRpc from "../rpc/ethRpc.vue";

export default Vue.extend({
  name: "BeginnerExampleMode",
  data() {
    return {
      loading: false,
      loginButtonStatus: "",
      connected: false,
      provider: undefined,
      namespace: undefined,
      web3auth: new Web3AuthCore({ chainConfig: { chainNamespace: CHAIN_NAMESPACES.EIP155 }, clientId: config.clientId["testnet"] }),
    };
  },
  components: {
    Loader,
    EthRpc,
  },
  async mounted() {
    await this.initWeb3Auth();
  },
  methods: {
    async initWeb3Auth() {
      try {
        this.web3auth = new Web3AuthCore({
          chainConfig: { chainId: "0x3", chainNamespace: CHAIN_NAMESPACES.EIP155 },
          clientId: config.clientId["testnet"],
        });
        this.subscribeAuthEvents(this.web3auth);
        const openloginAdapter = new OpenloginAdapter({
          adapterSettings: {
            network: "testnet",
            clientId: config.clientId["testnet"],
            uxMode: "redirect",
          },
        });

        this.web3auth.configureAdapter(openloginAdapter);
        await this.web3auth.init();
      } catch (error) {
        console.log("error", error);
        this.console("error", error);
      }
    },
    subscribeAuthEvents(web3auth: Web3AuthCore) {
      web3auth.on(ADAPTER_STATUS.CONNECTED, (data: CONNECTED_EVENT_DATA) => {
        this.console("connected to wallet", data);
        this.provider = web3auth.provider;
        this.loginButtonStatus = "Logged in";
        this.connected = true;
      });
      web3auth.on(ADAPTER_STATUS.CONNECTING, () => {
        this.console("connecting");
        this.loginButtonStatus = "Connecting...";
      });
      web3auth.on(ADAPTER_STATUS.DISCONNECTED, () => {
        this.console("disconnected");
        this.loginButtonStatus = "";
        this.connected = false;
      });
      web3auth.on(ADAPTER_STATUS.ERRORED, (error) => {
        console.log("error", error);
        this.console("errored", error);
        this.loginButtonStatus = "";
      });
    },
    async connect(e) {
      try {
        this.provider = await this.web3auth.connectTo(WALLET_ADAPTERS.OPENLOGIN, {
          loginProvider: e.target.name,
          login_hint: "",
        } as OpenloginLoginParams);
      } catch (error) {
        console.error(error);
        this.console("error", error);
      }
    },

    async logout() {
      await this.web3auth.logout();
      this.provider = undefined;
    },
    async getUserInfo() {
      const userInfo = await this.web3auth.getUserInfo();
      this.console(userInfo);
    },
    console(...args: unknown[]): void {
      const el = document.querySelector("#console>p");
      if (el) {
        el.innerHTML = JSON.stringify(args || {}, null, 2);
      }
    },
  },
});
</script>
