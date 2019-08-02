<template>
  <div id="app">
    <MockRuleMgmt/>
  </div>
</template>

<script>
import MockRuleMgmt from './components/MockRuleMgmt.vue'

import Vue from "vue";
import vuetabs from "vue-nav-tabs";
import "iview/dist/styles/iview.css";
import iview from "iview";
import axios from "axios";
import VueAxios from "vue-axios";

Vue.use(VueAxios, axios);
Vue.use(iview);



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
