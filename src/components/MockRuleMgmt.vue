<template>
  <div>
    <Divider>Mock 规则管理</Divider>
    <Input class="input" v-model="hostName" placeholder="hostName: * 表示可以匹配所有hostName" style="width: 300px" />
    <Input class="input" v-model="requestUri" placeholder="uri: uri 以/为开头" style="width: 300px" />
    <Button class="button" type="primary" @click="queryMockRules()">查询</Button>
    <Button class="button" type="primary" @click="addMockRule()">添加</Button>
    <noticeinformation ref="noticeinformation"></noticeinformation>
    <Divider orientation="left">Mock规则列表</Divider>
    <!--Mock 规则列表表格 -->
    <Table border :columns="columns" :data="data"></Table>
    <Page class="page" :total="mockRulesTotalSize" show-total show-sizer :page-size-opts="[10,20,30]" :page-size="10" @on-change="changePageNumber($event)"  @on-page-size-change="changePageSize($event)"/>
    <Card :bordered="true">
      <p slot="title">mock匹配规则说明</p>
      <p>1. hostName和uri作为联合索引, 即同样的hostName和uri 只能添加一条</p>
      <p>2. hostName 为* , 即表示可以匹配所有的hostName.</p>
      <p>3. hostName(不为*) 和 uri如果 没有匹配到,会尝试匹配hostName为*的规则</p>
      <p></p>
      <p>
        示例如下:
        若添加有2条规则
      </p>
      <p>第一条规则: hostName:* uri:/hello mockResponse: mock1</p>
      <p>第二条规则: hostName:testHostName uri:/hello mockResponse: mock2</p>

      <p>当我访问 http://ip/hello 时(ip是通过ip地址访问mock server), 则会匹配第一条规则.</p>
      <p>如果我访问者 http://testHostName/hello 时,会匹配到第二条规则. (testHostName 需要添加hosts 或者dns添加record)</p>
      <p>再如果我访问 http://testanotherHostName/hello 时, 会尝试先匹配 hostName=testanotherHostName, uri=/hello 如果找不到则会寻找</p>
      <p>hostName=testanotherHostName , uri=/ , 如果找不到则开始寻找 hostName=* , uri=/hello .</p>
    </Card>

    <Modal
      width="600px"
      v-model="addRuleModal"
      :title="modalTitle"
      @on-ok="addOk"
      @on-cancel="cancel"
    >
      <div>
        <span class="modalInputLabel">Host:</span>
        <Input
          v-model="addRule.host"
          placeholder="要添加的HostName,为空或者* 则表示会匹配所有HostName"
          style="width: 400px"
        />
      </div>
      <div>
        <span class="modalInputLabel">工作模式:</span>
        <Select v-model="addRule.workMode" style="width:200px">
          <Option v-for="item in workModes" :value="item.value" :key="item.value">{{ item.label }}</Option>
        </Select>
      </div>
      <div>
        <span class="modalInputLabel">Uri:</span>
        <Input v-model="addRule.uri" placeholder="要匹配的Uri路径,不含协议地址和端口号,以/开头" style="width: 400px" />
      </div>
      <div>
        <span class="modalInputLabel">Mock报文:</span>
        <Input
          v-model="addRule.mockResponse"
          type="textarea"
          :rows="10"
          placeholder="mock response body"
        />
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
      hostName: "",
      requestUri: null,
      mockRulesTotalSize:0,
      pageSize:10,
      pageNumber:1,
      modalTitle: "",
      workModes: [
        { value: "MOCK", label: "Mock" },
        { value: "UPSTREAM", label: "上游" }
      ],
      addRule: {
        enable:true,
        id: null,
        host: '*',
        uri: null,
        mockResponse: null,
        workMode: 'MOCK',
        update: false
      },
      update: null,
      addRuleModal: false,
      deleteRuleModal: false,
      columns: [
        {
          title: "id",
          key: "id",
          // render: (h, params) => {
          //   return h("div", params.row._id.$oid);
          // }
        },
        {
          title: "hostName",
          key: "host"
        },
        {
          title: "uri",
          key: "uri"
        },
        {
          title: "工作模式",
          key: "workMode"
        },
        {
          title: "mock报文",
          key: "mockResponse"
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
      this.pageSize = size
      this.queryMockRules()
    },
    changePageNumber: async function(number){
      this.pageNumber = number
      this.queryMockRules()

    },
    queryMockRules: async function() {
      let uri = this.server + "/api/mock/2.0/queryRule" + ""

      let requestBody = { host: this.hostName, uri: this.requestUri , pageNumber:this.pageNumber -1 ,pageSize:this.pageSize}

      let postresult = await this.axios.post(uri, requestBody)

      console.log(postresult.data.data)
      if(postresult.data.data != null)
      {
        this.mockRulesTotalSize = postresult.data.data.totalElements
        this.data = postresult.data.data.content
      }
      else{
         this.data  = []
      }

      this.$refs.noticeinformation.clear()
    },

    addMockRule: async function() {
      this.addRule.id = null
      this.addRule.update = false;
      this.addRule.uri = null;
      this.addRule.mockResponse = null;
      this.modalTitle = "添加Mock规则";
      this.addRuleModal = true;
    },
    deleteMockRule: async function(params) {
      this.addRule.host = params.row.host;
      this.addRule.uri = params.row.uri;
      this.addRule.mockResponse = params.row.mockResponse;
      //console.log(params.row._id);
      this.addRule.id = params.row.id;
      this.addRule.update = true;
      this.deleteRuleModal = true;

      //not supported
    },
    updateMockRule: async function(params) {
      this.addRule.host = params.row.host;
      this.addRule.uri = params.row.uri;
      this.addRule.mockResponse = params.row.mockResponse;
      this.addRule.id = params.row.id
      this.addRule.update = true;
      this.modalTitle = "修改Mock规则";
      this.addRuleModal = true;
    },
    copyMockRule: async function(params) {
      this.addRule.host = params.row.host;
      this.addRule.id = null
      this.addRule.uri = params.row.uri;
      this.addRule.mockResponse = params.row.mockResponse;
      this.addRule.update = false;
      this.modalTitle = "根据已有规则创建Mock规则";
      this.addRuleModal = true;
    },
    addOk: async function() {
      let uri;

      if (!this.addRule.update) uri = this.server + "/api/mock/2.0/addRule" + "";
      else uri = this.server + "/api/mock/2.0/updateRule" + "";

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
      let uri = this.server + "/api/mock/2.0/deleteRule" + "";
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
.page{
  margin:10px 10px 30px;
}
.input , .button{
  
  margin:0px 5px;
}
</style>
