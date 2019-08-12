<template>
  <div>
    <Divider>Eureka Mock Rule 管理</Divider>
    <Input
      class="input"
      v-model="eurekaServerAddress"
      placeholder="Eureka Server Address"
      style="width: 300px"
    />
    <Input class="input" v-model="serviceName" placeholder="Service Name" style="width: 300px" />
    <Button class="button" type="primary" @click="queryMockRules()">查询</Button>
    <Button class="button" type="primary" @click="addMockRule()">添加</Button>
    <noticeinformation ref="noticeinformation"></noticeinformation>
    <!--Mock 规则列表表格 -->
    <Table border :columns="columns" :data="data"></Table>
    <Page
      class="page"
      :total="mockRulesTotalSize"
      show-total
      show-sizer
      :page-size-opts="[10,20,30]"
      :page-size="10"
      @on-change="changePageNumber($event)"
      @on-page-size-change="changePageSize($event)"
    />

    <Card :bordered="true">
      <p slot="title">eureka mock规则说明</p>
    </Card>

    <Modal
      width="600px"
      v-model="addRuleModal"
      :title="modalTitle"
      @on-ok="addOk"
      @on-cancel="cancel"
    >
      <div>
        <span class="modalInputLabel">HostName:</span>
        <Input v-model="addRule.hostName" placeholder="要添加的HostName,不能为空" style="width: 400px" />
      </div>

      <div>
        <span class="modalInputLabel">Port:</span>
        <Input v-model="addRule.port" placeholder="端口号" style="width: 400px" />
      </div>

      <div>
        <span class="modalInputLabel">serviceName:</span>
        <Input v-model="addRule.serviceName" placeholder="serviceName" style="width: 400px" />
      </div>

      <div>
        <span class="modalInputLabel">eurekaServer:</span>
        <Input v-model="addRule.eurekaServer" placeholder="eurekaServer" style="width: 400px" />
      </div>
    </Modal>

    <Modal
      width="600px"
      v-model="deleteRuleModal"
      title="删除Mock规则"
      @on-ok="deleteOk"
      @on-cancel="cancel"
    >
      <div>
        将删除规则:
        [ hostName: {{addRule.host}} ]
        [ uri: {{addRule.uri}} ]
      </div>
    </Modal>
  </div>
</template>
<script>
export default {
  created: function() {
    this.queryMockRules();
  },
  data() {
    return {
      //server: "http://172.16.2.39:7280",
      server: "http://localhost:8081",
      eurekaServerAddress: null,
      serviceName: null,
      mockRulesTotalSize: 0,
      pageSize: 10,
      pageNumber: 1,
      modalTitle: "",
      addRule: {
        enable: true,
        id: null,
        port: "",
        hostName: "",
        serviceName: "",
        eurekaServer: "",
        update: false
      },
      update: null,
      addRuleModal: false,
      deleteRuleModal: false,
      columns: [
        {
          title: "id",
          key: "id"
          // render: (h, params) => {
          //   return h("div", params.row._id.$oid);
          // }
        },
        {
          title: "eurekaServer",
          key: "eurekaServer"
        },
        {
          title: "hostName",
          key: "hostName"
        },
        {
          title: "port",
          key: "port"
        },
        {
          title: "serviceName",
          key: "serviceName"
        },

        {
          title: "操作",
          key: "action",
          width: 300,
          align: "center",
          render: (h, params) => {
            return h("div", [
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.deleteMockRule(params);
                    }
                  }
                },
                "删除"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.updateMockRule(params);
                    }
                  }
                },
                "修改"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "primary",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.copyMockRule(params);
                    }
                  }
                },
                "复制创建"
              )
            ]);
          }
        }
      ],
      data: []
    };
  },
  methods: {
    changePageSize: async function(size) {
      this.pageSize = size;
      this.queryMockRules();
    },
    changePageNumber: async function(number) {
      this.pageNumber = number;
      this.queryMockRules();
    },
    queryMockRules: async function() {
      let uri = this.server + "/api/mock/2.0/queryEurekaRule" + "";

      let requestBody = {
        eurekaServer: this.eurekaServerAddress,
        serviceName: this.serviceName,
        pageNumber: this.pageNumber - 1,
        pageSize: this.pageSize
      };

      let postresult = await this.axios.post(uri, requestBody);

      console.log(postresult.data.data);
      if (postresult.data.data != null) {
        this.mockRulesTotalSize = postresult.data.data.totalElements;
        this.data = postresult.data.data.content;
      } else {
        this.data = [];
      }

      this.$refs.noticeinformation.clear();
    },

    addMockRule: async function() {
      this.addRule.id = null;
      this.addRule.update = false;
      this.addRule.port = null
      this.addRule.hostName = null
      this.addRule.serviceName = null
      this.addRule.eurekaServer = null
      this.modalTitle = "添加Mock规则";
      this.addRuleModal = true;
    },
    deleteMockRule: async function(params) {
      this.addRule = params.row;
      //console.log(params.row._id);
      this.addRule.update = true;
      this.deleteRuleModal = true;
      //not supported
    },
    updateMockRule: async function(params) {
      this.addRule = params.row;
      this.addRule.update = true;
      this.modalTitle = "修改Mock规则";
      this.addRuleModal = true;
    },
    copyMockRule: async function(params) {
      this.addRule = params.row;
      this.addRule.id = null
      //console.log(params.row._id);
      this.addRule.update = false;
      this.modalTitle = "根据已有规则创建Mock规则";
      this.addRuleModal = true;
    },
    addOk: async function() {
      let uri;

      if (!this.addRule.update)
        uri = this.server + "/api/mock/2.0/addEurekaRule" + "";
      else uri = this.server + "/api/mock/2.0/updateEurekaRule" + "";

      //let requestBody = {'hostName':this.hostName,'uri':this.requestUri}

      let postresult = await this.axios.post(uri, this.addRule);

      console.log(postresult.data.data);

      if (postresult.data.success) {
        await this.queryMockRules();
        this.$refs.noticeinformation.showalert("success", "添加/修改成功");
      } else {
        this.$refs.noticeinformation.showalert(
          "error",
          "添加/修改失败,请确认参数是否填写正确 或者 同样的hostName和uri规则是否已经添加!"
        );
      }
    },
    deleteOk: async function() {
      let uri = this.server + "/api/mock/2.0/deleteEurekaRule" + "";
      let postresult = await this.axios.post(uri, { id: this.addRule.id });
      if (postresult.data.success) {
        await this.queryMockRules();
        this.$refs.noticeinformation.showalert("success", "删除成功");
      } else {
        this.$refs.noticeinformation.showalert("error", "删除失败!");
      }
    },
    cancel: async function() {}
  }
};
</script>
<style scoped>
.modalInputLabel {
  width: 80px;
  padding: 10px;
  display: inline-block;
}
p {
  text-align: left;
}
.page {
  margin: 10px 10px 30px;
}
.input,
.button {
  margin: 0px 5px;
}
</style>
