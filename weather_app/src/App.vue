<template>
  <img alt="Logo" src="./assets/source.png" class="custom-img">
  <LandingPage></LandingPage>
</template>

<script>
import LandingPage from './components/LandingPage.vue'

export default {
  name: 'App',
  components: {
    LandingPage  
  }
}
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
.custom-img {
    width: 200px;
    height: auto;
  }

</style>
