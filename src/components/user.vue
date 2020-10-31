<template>
  <div>
<div class="loginform">
      <div>
        <span class="modalInputLabel">username:</span>
        <Input
          v-model="username"
          placeholder="username"
          style="width: 400px"
        />
      </div>

<div>
        <span class="modalInputLabel">password:</span>
        <Input
          type = "password"
          v-model="password"
          placeholder="password"
          style="width: 400px"
        />
      </div>

    <Button class="button" type="primary" @click="login">查询</Button>


</div>

<!-- class="usermgmt" start  -->
<div class="usermgmt">


</div>



</div>
</template>
<script>


export default {
  created: function() {

  },
  data() {
    return {
      username:'',
      password:'',
      server: this.$store.getters.getServer,
    }
  },
  methods: {
    login:async function(){

      console.log(document.domain)

      let uri

      uri = this.server + "/api/mock/2.0/login"

      //let requestBody = {'hostName':this.hostName,'uri':this.requestUri}

      let postresult = await this.axios.post(uri, {username:this.username, password:this.password}, {withCredentials: true})

      console.log(postresult.data.data)

      if (postresult.data.success) {
        //
      } else {
          //
      }

    }
  }
}
</script>
<style scoped>
</style>

<style>
</style>
