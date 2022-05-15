<template>
  <Header></Header>
  <main>
    <ShellySquare v-for="ip in ips" :key="ip" :shellyIP="ip"></ShellySquare>
  </main>
</template>

<script setup>
  import Header from './components/Header.vue';
  import ShellySquare from './components/ShellySquare.vue';
  import ips from './assets/IPs.json';
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    margin-top: 60px;
  }

  main {
    display: flex;
    flex-flow: row wrap;
  }

  main > div {
    box-shadow: rgba(0, 0, 0, 0.2) 5px 5px 10px 0px;
    margin: 1%;
    padding: 1%;
    border-radius: 15px;
  }
</style>
