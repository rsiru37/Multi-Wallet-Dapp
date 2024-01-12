<script setup>
import { createWeb3Modal, defaultConfig } from '@web3modal/ethers/vue'
import { useWeb3ModalAccount } from '@web3modal/ethers/vue'
import { watch } from 'vue';
import { Alchemy, Network } from "alchemy-sdk";
import {ref} from 'vue';
import axios from 'axios';
import {Connection,PublicKey,clusterApiUrl,Keypair,LAMPORTS_PER_SOL} from "@solana/web3.js";
const owner = ref();
const tokens = ref();
const status = ref(false);
const chain = ref();
const soladdress = ref();
const solana_tokens = ref();
const sbalanace = ref();
const projectId = '6e460449aaf08ef8388ee11ac4e3ce00';
const mainnet = {
  chainId: 1,
  name: 'Ethereum',
  currency: 'ETH',
  explorerUrl: 'https://etherscan.io',
  rpcUrl: 'https://cloudflare-eth.com'
};
const arbi = {
  chainId: 42161,
  name:'Arbitrum One',
  currency:'ETH',
  explorerUrl:"https://arbiscan.io",
  rpcUrl: 'https://arb1.arbitrum.io/rpc'
};
const matic = {
  chainId: 137,
  name:'Polygon',
  currency:'MATIC',
  explorerUrl:"https://polygonscan.com",
  rpcUrl: "https://polygon-rpc.com"
};
const bsc = {
  chainId: 56,
  name:'BNB Smart Chain',
  currency:'BNB',
  explorerUrl:"https://bscscan.com",
  rpcUrl: "https://rpc.ankr.com/bsc"
};
const op = {
  chainId:10,
  name:'OP Mainnet',
  currency:'ETH',
  explorerUrl:"https://explorer.optimism.io",
  rpcUrl:"https://opt-mainnet.g.alchemy.com/v2"
}
const chains = [mainnet, arbi,matic,bsc,op];
const wagmiConfig = defaultConfig({ chains, projectId});
createWeb3Modal({ ethersConfig: wagmiConfig, projectId, chains, themeMode: 'light', themeVariables: { '--w3m-color-mix':"#000000"}});
const { address, chainId, isConnected } = useWeb3ModalAccount();
console.log(useWeb3ModalAccount());

  const gwi = () => {
    owner.value = address.value;
    console.log("Con",address.value);
    status.value = true;
  }

  async function network(){
    switch(chainId.value){
      case 1 :
        chain.value = "eth-mainnet";
        break;
      case 42161:
        chain.value = "arb-mainnet";
        break;
      case 137:
        chain.value = "polygon-mainnet";
        break;
      case 10:
        chain.value = "opt-mainnet";
        break;
      default:
        chain.value = "eth-mainnet";
        break;
    }
    // alchemy.value = new Alchemy({ apiKey: "2Ff_bepC2VkMOS8MPLeKCA2PAUWg2X60",network: chain.value});
    console.log("Network changed", chainId.value);
    const alchemy = new Alchemy({ apiKey: "2Ff_bepC2VkMOS8MPLeKCA2PAUWg2X60",network: chain.value});
    let response = await alchemy.core.getTokensForOwner(owner.value);
    tokens.value = response.tokens;

  }
  async function connected(){
    if(isConnected){
      const alchemy = new Alchemy({ apiKey: "2Ff_bepC2VkMOS8MPLeKCA2PAUWg2X60",network: chain.value});
      let response = await alchemy.core.getTokensForOwner(owner.value);
      tokens.value = response.tokens;
      console.log("Tokens", tokens.value);
      console.log(Network);
    }
    else{
      status.value = false;
    }

  }
  watch(
    () => address.value,
    () => {
      gwi();
    }); 
  
    watch(
      () => chainId.value,
      () => {
        network();
      }
    );
    watch(
      () => isConnected.value,
      () => {
        connected();
      }
    )

  if(isConnected){
    owner.value = address.value;
    //let response = await alchemy.core.getTokensForOwner(owner);
    //console.log(response);
  }




async function sol(){
  console.log("Entered");
  //const { solana } = window;
  if(window.solana){
    const solana = window.solana;
    const response = await solana.connect();
    soladdress.value = response.publicKey.toString();
    console.log("PKey", response.publicKey);
    const connection = new Connection(clusterApiUrl("devnet"), "confirmed");
    sbalanace.value = await connection.getBalance(response.publicKey)/LAMPORTS_PER_SOL;
    console.log("Solana Balance", sbalanace);
    console.log("P",soladdress.value);
    let res = await axios.get(`https://api.shyft.to/sol/v1/wallet/all_tokens?network=devnet&wallet=${soladdress.value}`, 
      { headers: {
        "Content-Type": "application/json",
        "x-api-key": "4FZtkEfIooItoeQz",
        }
      });
      //console.log("S", solbalance.data);
    solana_tokens.value = res.data.result;
    console.log(res.data);
  }
}
</script>

<template>
    <header>
      <div class="header" style="margin: auto; text-align: center;">
      <h1 margin="auto">Multi Wallet Dapp</h1>
    </div>
  </header>
  <body>
    <div class="left-container">
        <!-- Content for the left container goes here -->
        <w3m-button/>
        <br>
        <div v-if="status">
          <h2>TOKENS :</h2>
          <div v-for="token in tokens" >
            <h5 style="display: inline;">{{ token.symbol }} : </h5> <p style="display: inline;">{{ token.balance }}</p>
            <br>
          </div>
        </div>
    </div>

    <div class="right-container">
        <!-- Content for the right container goes here -->
        <div v-if="!soladdress">
        <button type="button" class="btn btn-dark" @click="sol()">Connect Solana Wallet</button>
      </div>
      <div v-else>
        <button type="button" class="btn btn-success">Connected</button>
        <p> {{ soladdress }}</p>
        <h2>TOKENS :</h2>
          <h5>SOL : {{ sbalanace }}</h5>
          <div v-for="coin in solana_tokens" >
            <h5 style="display: inline;">{{ coin.address }} : </h5> <p style="display: inline;">{{ coin.balance }}</p>
            <br>
          </div>
      </div>
    </div>
  </body>

  <!-- <button type="button" class="btn btn-primary" v-on:click="connecter()">Connect Wallet</button> -->
</template>

<style>
        body {
          display: flex;
            height: 100vh;
            flex-direction: row;
        }

        .left-container, .right-container {
            flex: 1;
            height: 100%;
            box-sizing: border-box;
            padding: 20px;
        }

        .left-container {
            background-color: #f0f0f0;
        }

        .right-container {
            background-color: #e0e0e0;
        }
    </style>

