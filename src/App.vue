<template>
  <div id="app">
    <div class="header">MockServer Management

      <div class="rightMenu"> {{loginName}}</div>
    </div>

    
    <div id="body">

      <div>
      <Tabs type="card">
        <TabPane key="mockRule" name="mockRule" label="Http Mock Rule" class="show_align">
          <MockRuleMgmt />
        </TabPane>

        <TabPane
          key="eurekaMockRule"
          name="eurekaMockRule"
          label="Eureka Mock Rule"
          class="show_align"
        >
          <EurekaMockRuleMgmt />
        </TabPane>
      </Tabs>
      </div>

      <div>
        <usermgmt></usermgmt>
      </div>


    </div>



    <Divider></Divider>
    <div style="text-align:center;margin:20px;font-size:1.5em;">
      <a href="https://www.hissummer.com" target="_blank">Hissummer Mockserver</a> |
      <a href="https://mockserver.hissummer.com/" target="_blank">使用帮助</a>
    </div>


  </div>
</template>

<script>
import MockRuleMgmt from "./components/MockRuleMgmt.vue";
import EurekaMockRuleMgmt from "./components/EurekaMockRuleMgmt.vue";
import noticeinformation from "./components/noticeinformation.vue";

import Vue from "vue";
//import vuetabs from "vue-nav-tabs"
import "iview/dist/styles/iview.css";
import iview from "iview";
import axios from "axios";
import VueAxios from "vue-axios";
import Vuex from "vuex";

//import clonedeep from "lodash.clonedeep"

Vue.use(VueAxios, axios);
Vue.use(iview);
Vue.component("noticeinformation", noticeinformation);
Vue.use(Vuex);

import usermgmt from "./components/user.vue";
Vue.component("usermgmt", usermgmt);



const store = new Vuex.Store({
  state: {
    server: "http://localhost:8081",
    standalone: true 
  },
  getters: {
    getServer(state) {
      if (state.standalone) {
        return state.server;
      } else {
        return window.location.protocol + "//" + window.location.host;
      }
    }
  }
});

export default {
  name: "app",
  store,
  components: {
    MockRuleMgmt,
    EurekaMockRuleMgmt
  },
  data(){ 
    
    return {

        loginName:'noLogin'
  }

  }


};
</script>

<style scope>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#body {
  margin: 0px 20px;
}

.header {
  height: 60px;
  /* text-align: center; */
  /* vertical-align: middle; */
  padding: 20px;
  background-color: #3e8bf0;
  color: white;
  font-weight: 1000;
  font-size: 1.5em;
  margin-bottom: 10px;
}

.rightMenu{
  float:right;
  font-size:0.7em;
}

</style>

<style >

.ivu-tabs-card{
      padding-bottom: 78px;
}
</style>
