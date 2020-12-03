<template>
<div>

    <noticeinformation ref="noticeinformation"></noticeinformation>

    <div class="repassword">

        <div class="modalInputLabel">
            <span class="modalInputLabel">username:</span>
            <Input disabled v-model="username" placeholder="username" style="width: 400px" />
        </div>

        <div class="modalInputLabel">
            <span class="modalInputLabel">now password:</span>
            <Input type="password" v-model="password" placeholder="password" style="width: 400px" />
        </div>

        <div class="modalInputLabel">
            <span class="modalInputLabel">new password:</span>
            <Input type="password" v-model="newpassword" placeholder="password" style="width: 400px" />
        </div>

        <div class="modalInputLabel">
            <span class="modalInputLabel">new password repeat:</span>
            <Input type="password" v-model="passwordrepeat" placeholder="password" style="width: 400px" />
        </div>

        <Button class="button" type="primary" @click="login">confirm</Button>

    </div>

</div>
</template>

<script>
export default {
    created: function () {

    },
    data() {
        return {
            username: localStorage.username,
            password: '',
            newpassword: '',
            passwordrepeat: '',
            server: this.$store.getters.getServer,
            showloginform: true,
            newaxios: this.axios.create({
                withCredentials: true
            })
        }
    },
    methods: {

        login: async function () {

            if (!this.checknewpassword()) return

            let uri

            uri = this.server + "/api/mock/2.0/rePassword"

            //let requestBody = {'hostName':this.hostName,'uri':this.requestUri}

            let postresult = await this.newaxios.post(uri, {
                username: this.username,
                password: this.password,
                newpassword: this.newpassword
            })

            console.log(postresult)

            if (postresult.status == 200 && postresult.data.success) {

                //login success
                this.$refs.noticeinformation.showalert("success", "changed password ok!")

            } else {
                //
                this.$refs.noticeinformation.showalert("warning", "change password failed, please check username and password!")

            }

        },
        checknewpassword: function () {

            if (this.newpassword !== this.passwordrepeat) {
                this.$refs.noticeinformation.showalert("warning", "new password are not equal!")
                return false
            }

            return true

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
