<template>
<div>

    <noticeinformation ref="noticeinformation"></noticeinformation>

    <div v-if="showloginform" class="loginform">
        <div class="modalInputLabel">
            <span class="modalInputLabel">username:</span>
            <Input v-model="username" placeholder="username" style="width: 400px" />
        </div>

        <div class="modalInputLabel">
            <span class="modalInputLabel">password:</span>
            <Input type="password" v-model="password" placeholder="password" style="width: 400px" />
        </div>

        <Button class="button" type="primary" @click="login">Login</Button>

    </div>

</div>
</template>

<script>
export default {
    created: function () {

    },
    data() {
        return {
            username: '',
            password: '',
            server: this.$store.getters.getServer,
            showloginform: true,
            newaxios: this.axios.create({
                withCredentials: true
            })
        }
    },
    methods: {

        login: async function () {

            let uri

            uri = this.server + "/xxxxhissummerxxxx/api/login"

            //let requestBody = {'hostName':this.hostName,'uri':this.requestUri}

            let postresult = await this.newaxios.post(uri, {
                username: this.username,
                password: this.password
            })

            this.$log.debug(postresult)

            if (postresult.status == 200 && postresult.data.success) {

                //login success
                this.$refs.noticeinformation.clear()

                this.showloginform = false

                this.$emit('logon', this.username)
                localStorage.username = this.username

            } else {
                //
                this.$refs.noticeinformation.showalert("warning", "login failed!")

            }

        }
    }
}
</script>

<style scoped>
.button {
    margin: 10px 0px;
}

div.modalInputLabel {
    margin: 15px 5px;
}

span.modalInputLabel {
    margin-right: 15px;
}
</style>

<style>
</style>
