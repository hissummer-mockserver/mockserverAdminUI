<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <MockRuleMgmt/>
  </div>
</template>

<script>
import MockRuleMgmt from './components/MockRuleMgmt.vue'

export default {
  name: 'app',
  components: {
	MockRuleMgmt
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
