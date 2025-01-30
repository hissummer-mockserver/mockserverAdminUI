<template>
<div id="app">
    <div class="header">MockServer Management

        <div v-if="!showUserlogin " class="rightMenu">
            <Dropdown>

                <a style="color:white;">{{loginName}}</a>
                <DropdownMenu slot="list">
                    <DropdownItem v-if="!showTabs"><span @click="showtabs">rule Management</span></DropdownItem>
                    <DropdownItem><span @click="showchangePasswordForm">change Password</span></DropdownItem>
                    <DropdownItem><span @click="logout">logout</span></DropdownItem>

                </DropdownMenu>
            </Dropdown>

        </div>

    </div>

    <div id="body">

        <div v-if="showTabs">
            <Tabs type="card">
                <TabPane key="mockRule" name="mockRule" label="Http Mock Rule" class="show_align">
                    <MockRuleMgmt />
                </TabPane>

                <TabPane key="eurekaMockRule" name="eurekaMockRule" label="Eureka Mock Rule" class="show_align">
                    <EurekaMockRuleMgmt />
                </TabPane>
            </Tabs>
        </div>
        <div v-if="!showUserlogin && !showTabs && !showrepassword">
            <div>Loading...</div>
        </div>
        <div v-if="showUserlogin">
            <userlogin v-on:logon="logon($event)"></userlogin>
        </div>

        <div v-if="showrepassword">
            <reuserpassword></reuserpassword>
        </div>

    </div>

    <Divider></Divider>
    <div style="text-align:center;margin:20px;font-size:1.5em;">
        <a href="http://qa.heika.com" target="_blank">Mockserver v0.0.9</a> |
        <!-- <a href="https://mockserver.hissummer.com/" target="_blank">使用帮助</a> -->
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

import userlogin from "./components/userlogin.vue";
Vue.component("userlogin", userlogin);

import reuserpassword from "./components/reuserpassword.vue";
Vue.component("reuserpassword", reuserpassword);

const store = new Vuex.Store({
    state: {
        //server: "http://localhost:8082",
        server: "http://core-mockplatform.test.weicai.com.cn",
        standalone: false
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
    data() {

        return {

            loginName: 'noLogin',
            showTabs: false,
            showUserlogin: false,
            showrepassword: false,
            newaxios: this.axios.create({
                withCredentials: true
            }),
            server: this.$store.getters.getServer,

        }

    },

    created: function () {
        this.checkLogin()
    },
    methods: {

        checkLogin: async function () {

            try {
                let uri

                uri = this.server + "/xxxxhissummerxxxx/api/isLogin"

                //let requestBody = {'hostName':this.hostName,'uri':this.requestUri}

                let postresult = await this.newaxios.post(uri, {})
                this.$log.debug(this.Vue)
                this.$log.debug('------------------')
                this.$log.debug(postresult)

                if (postresult.status == 200 && postresult.data.success) {

                    this.showTabs = true
                    this.showUserlogin = false
                    this.loginName = 'welcome ' + localStorage.username + ''

                } else {
                    this.showTabs = false
                    this.showUserlogin = true

                }
            } catch (e) {
                this.$log.debug(e)
                this.$log.debug(e.response)
                this.showTabs = false
                this.showUserlogin = true
            }

        },

        logout: async function () {

            let uri

            uri = this.server + "/xxxxhissummerxxxx/api/logout"

            //let requestBody = {'hostName':this.hostName,'uri':this.requestUri}

            let postresult = await this.newaxios.post(uri, {
                username: localStorage.username
            })

            this.$log.debug(postresult)

            if (postresult.status == 200 && postresult.data.success) {

                this.showTabs = false
                this.showrepassword = false
                this.showUserlogin = true
                this.loginName = 'noLogin'

            } else {

                this.checkLogin()
            }

        },
        logon: function (username) {

            this.showTabs = true
            this.showUserlogin = false
            this.loginName = 'welcome ' + username + ''

        },
        showchangePasswordForm: function () {

            this.showTabs = false
            this.showrepassword = true

        },
        showtabs: function () {
            this.showTabs = true
            this.showrepassword = false

        }
    }

};
</script>

<style>
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

.rightMenu {
    float: right;
    font-size: 0.7em;
    cursor: pointer;
    margin: 10px 0px 5px;
}

.rightMenu span {

    width: auto;
    padding: 3px 10px;
    display: block !important;

}

.loading {
    font-size: 2em;
    font-weight: bold;
}
</style><style>
.ivu-tabs-card {
    padding-bottom: 78px;
}
.modalInputLabel {
    width:200px;
  padding: 10px;
  font-weight:700;
}
</style>
