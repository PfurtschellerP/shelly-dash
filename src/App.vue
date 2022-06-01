<template>
  <Header></Header>
  <main>
    <ShellySquare v-for="ip in ips" :key="ip" :shellyIP="ip"></ShellySquare>
  </main>
  <Footer></Footer>
</template>

<script setup>
  import Header from './components/Header.vue';
  import ShellySquare from './components/ShellySquare.vue';
  import Footer from './components/Footer.vue';
  import ips from './assets/IPs.json';
</script>

<style>
  :root {
    --background-color: rgba(0, 0, 0, 0.1);

    --header-bg-color: rgba(0, 0, 0, 0.5);
    --header-text-color: white;

    --footer-bg-color: rgba(0, 0, 0, 0.5);
    --footer-text-color: white;
  }

  * {
    margin: 0;
    padding: 0;
  }

  body {
    background-color: var(--background-color);
  }

  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  main {
    display: grid;
    gap: 50px;
    grid-template-columns: 1fr 1fr 1fr;
    padding: 0 2vw 2vh;
    color: #2c3e50;
  }

  main > div {
    background-color: white;
    box-shadow: rgba(0, 0, 0, 0.2) 5px 5px 10px 0px;
    padding: 2%;
    border-radius: 15px;
    transition: all 0.2s ease-in-out;
  }

  main > div:hover {
    transform: scale(1.05);
    cursor: pointer;
  }

  @media only screen and (max-width: 1000px) {
    main {
      grid-template-columns: 1fr 1fr;
    }
  }

  @media only screen and (max-width: 750px) {
    main {
      grid-template-columns: 1fr;
    }
  }
</style>
