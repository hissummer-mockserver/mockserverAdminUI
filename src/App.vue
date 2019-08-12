<template>
<div id="app">
  <div class="header">Hissummer Mock Server</div>
  <div id="body">
          <Tabs type="card" v-model="activeTab" >
        <TabPane
          key="mockRule"
          name="mockRule"
          label="Http Mock Rule"
          class="show_align"
        >
           <MockRuleMgmt/>
        </TabPane>

        <TabPane
          key="eurekaMockRule"
          name="eurekaMockRule"
          label="Eureka Mock Rule"
          class="show_align"
        >
           <EurekaMockRuleMgmt/>
        </TabPane>

      </Tabs>
   
   </div>
    <div style="text-align:center;margin:20px;font-size:1.5em;"><a href="http://www.hissummer.com">www.hissummer.com</a></div>
  
  </div>

</template>

<script>
import MockRuleMgmt from './components/MockRuleMgmt.vue'
import EurekaMockRuleMgmt from './components/EurekaMockRuleMgmt.vue'
import noticeinformation from './components/noticeinformation.vue'

import Vue from "vue"
import vuetabs from "vue-nav-tabs"
import "iview/dist/styles/iview.css"
import iview from "iview"
import axios from "axios"
import VueAxios from "vue-axios"

Vue.use(VueAxios, axios)
Vue.use(iview)
Vue.component('noticeinformation',noticeinformation)



export default {
  name: 'app',
  components: {
	MockRuleMgmt,EurekaMockRuleMgmt
  }


}
</script>

<style scope>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#body{
  margin:0px 20px;
}

.header{
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

</style>
