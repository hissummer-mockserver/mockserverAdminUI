<template>
  <div>
    <Divider>Http Mock Rule 管理</Divider>
    <Input
      class="input"
      v-model="hostName"
      placeholder="hostName: 例如*"
      style="width: 300px"
    />
    <Input class="input" v-model="requestUri" placeholder="uri: uri 以/为开头" style="width: 300px" />
    <Button class="button" type="primary" @click="changePageNumber(1)">查询</Button>
    <Button class="button" type="primary" @click="addMockRule()">添加</Button>
    <noticeinformation ref="noticeinformation"></noticeinformation>
    <!--Mock 规则列表表格 -->
    <Table border :columns="columns" :data="data"></Table>
    <Page
      class="page"
      :total="mockRulesTotalSize"
      :current="pageNumber"
      show-total
      show-sizer
      :page-size-opts="[10,20,30]"
      :page-size="10"
      @on-change="changePageNumber($event)"
      @on-page-size-change="changePageSize($event)"
    />

    <!-- start of the add modal -->
    <Modal
      width="600px"
      v-model="addRuleModal"
      :title="modalTitle"
      @on-ok="addOk"
      @on-cancel="cancel"
    >
      <div>
        <span class="modalInputLabel">请求Host:</span>
        <Input
          v-model="addRule.host"
          placeholder="要添加的HostName,为空或者* 则表示会匹配所有HostName"
          style="width: 400px"
        />
      </div>
      <div>
        <span class="modalInputLabel">工作模式:</span>
        <Select v-model="addRule.workMode" style="width:200px" @on-change="changeWorkMode($event)">
          <Option v-for="item in workModeOption" :value="item.value" :key="item.value" >{{ item.label }}</Option>
        </Select>
        <div v-if="showUpstreamMode">
        <div class="nextedForm">
          <span class="modalInputLabel">上游服务: </span>

          <Input
            v-model="addRule.upstreams.nodes[0].protocol"
            placeholder="http"
            style="width:50px"
          /> ://
          <Input
            v-model="addRule.upstreams.nodes[0].address"
            placeholder="localhost:8081"
            style="width:150px"
          />
          <Input
            v-model="addRule.upstreams.nodes[0].uri"
            placeholder="/testuri"
            style="width:150px"
          />

        </div>

        </div> 

      </div>

      <div>
        <span class="modalInputLabel">请求Uri:</span>
        <Input v-model="addRule.uri" placeholder="要匹配的Uri路径,不含协议地址和端口号,以/开头" style="width: 400px" />
      </div>

      <div v-if="!showUpstreamMode">
        <span class="modalInputLabel">响应Headers:</span>
        <Input
          v-model="addRule.responseHeaders"
          type="textarea"
          :rows="5"
          placeholder="{'header1':'value1'}"
        />
      </div>

      <div  v-if="!showUpstreamMode">
        <span class="modalInputLabel">响应Mock报文:</span>
        <Input
          v-model="addRule.mockResponse"
          type="textarea"
          :rows="10"
          placeholder="mock response body"
        />
      </div>
    </Modal>

    <!-- end of the add modal -->

    <!-- start of the delete modal -->
    <Modal
      width="600px"
      v-model="deleteRuleModal"
      title="删除Mock规则"
      @on-ok="deleteOk"
      @on-cancel="cancel"
    >
      <div>
        <span class="modalInputLabel">请求Host:</span>
        <Input disabled 
          v-model="addRule.host"
          placeholder="要添加的HostName,为空或者* 则表示会匹配所有HostName"
          style="width: 400px"
        />
      </div>
      <div>
        <span class="modalInputLabel">工作模式:</span>
        <Select disabled  v-model="addRule.workMode" style="width:200px" @on-change="changeWorkMode($event)">
          <Option v-for="item in workModeOption" :value="item.value" :key="item.value" >{{ item.label }}</Option>
        </Select>
        <div v-if="showUpstreamMode">
        <div class="nextedForm">
          <span class="modalInputLabel">上游服务: </span>

          <Input disabled 
            v-model="addRule.upstreams.nodes[0].protocol"
            placeholder="http"
            style="width:50px"
          /> ://
          <Input disabled 
            v-model="addRule.upstreams.nodes[0].address"
            placeholder="localhost:8081"
            style="width:150px"
          />
          <Input disabled 
            v-model="addRule.upstreams.nodes[0].uri"
            placeholder="/testuri"
            style="width:150px"
          />

        </div>

        </div> 

      </div>

      <div>
        <span class="modalInputLabel">请求Uri:</span>
        <Input disabled v-model="addRule.uri"  style="width: 400px" />
      </div>

      <div v-if="!showUpstreamMode">
        <span class="modalInputLabel">响应Headers:</span>
        <Input disabled 
          v-model="addRule.responseHeaders"
          type="textarea"
          :rows="3"

        />
      </div>

      <div  v-if="!showUpstreamMode">
        <span class="modalInputLabel">响应Mock报文:</span>
        <Input disabled 
          v-model="addRule.mockResponse"
          type="textarea"
          :rows="5"
        />
      </div>
    </Modal>

    <!-- end of the delete modal -->

  </div>
</template>
<script>
//var _ = require('lodash')

import lodash from "lodash"

export default {
  created: function() {
    this.queryMockRules()
  },
  data() {
    return {
      server: this.$store.getters.getServer,
      hostName: "",
      requestUri: null,
      mockRulesTotalSize: 0,
      pageSize: 10,
      pageNumber: 1,
      modalTitle: "",
      workModeOption: [
        { value: "MOCK", label: "Mock" },
        { value: "UPSTREAM", label: "Upstream" }
      ],
      showUpstreamMode:false,
      addRule: {
        enable: true,
        id: null,
        host: "*",
        uri: null,
        mockResponse: null,
        workMode: "MOCK",
        upstreams:{nodes:[{protocol:'',address:'',uri:''}]},
        update: false,
        responseHeaders: null
      },
      update: null,
      addRuleModal: false,
      deleteRuleModal: false,
      columns: [
        {
          title: "id",
          key: "id",
          width:120
          // render: (h, params) => {
          //   return h("div", params.row._id.$oid)
          // }
        },
        {
          title: "hostName",
          key: "host",
          width:250
        },
        {
          title: "uri",
          key: "uri"
        },
        {
          title: "工作模式",
          width: 100,
          key: "workMode"
        },
        {
          title:"上游节点",
          render: this.renderUpstreamNodes,
          width:180
        },
        {
          title: "响应Headers",          
          width:150,
          render: this.renderMockResponseHeaders
        },
        {
          title: "响应mock报文",
          key: "mockResponse",
          render: this.renderMockResponseColumn
        },
        {
          title: "操作",
          key: "action",
          width: 200,
          align: "center",
          render: this.renderActionColumn
        }
      ],
      data: []
    }
  },
  methods: {

    renderUpstreamNodes:function(h,params){
      let upstreamNodes = 'N/A'
      if(params.row.workMode == 'UPSTREAM' )
      {
        upstreamNodes = params.row.upstreams.nodes
      }

      return h("div",this.showless(upstreamNodes))
    },

    renderMockResponseColumn:function(h,params){

      let mockResponse = 'N/A'
      if(params.row.workMode == 'MOCK' )
      {
        mockResponse = params.row.mockResponse == null ? '无': params.row.mockResponse
              return h(
                "Tooltip",
                {
                  props: {transfer:true,theme:'light',placement: "left-start", 'max-width':'500',content: mockResponse}
                },
                [
                  h(
                    "div",
                  this.showless(mockResponse)
                  )
                ]
              )
      }
      else{
       return  h("div",mockResponse)
      }


    },

    renderMockResponseHeaders: function(h,params){
        let mockResponseHeaders = 'N/A'
        if(params.row.workMode == 'MOCK' )
        {
          mockResponseHeaders = params.row.responseHeaders == null ? '无': params.row.responseHeaders


        }
        return h("div",this.showless(mockResponseHeaders))
    }
    ,
    renderActionColumn:function(h,params){
      
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
                      this.deleteMockRule(params)
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
                      this.updateMockRule(params)
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
                      this.copyMockRule(params)
                    }
                  }
                },
                "复制创建"
              )
            ])
    },

    changeWorkMode:function(workMode){

      if(workMode == 'UPSTREAM')
      {
        this.showUpstreamMode = true
        this.responseBody = false
        this.responseHeaders = false
      }
      else{
        this.showUpstreamMode = false
        this.addRule.upstreams = {nodes:[{protocol:'',address:'',uri:''}]}
        this.responseBody = true
        this.responseHeaders = true
      }

    },

    showless: function(message) {
        
        let showlessMessage = typeof message === 'string' || message instanceof String ? message : JSON.stringify(message)

        return  showlessMessage.length > 128
                    ? showlessMessage.substring(0, 128) + " ...更多内容"
                    : showlessMessage
    },

    changePageSize: async function(size) {
      this.pageSize = size
      this.queryMockRules()
    },
    changePageNumber: async function(number) {
      this.pageNumber = number
      this.queryMockRules()
    },
    queryMockRules: async function() {
      let uri = this.server + "/api/mock/2.0/queryRule" + ""

        let requestBody = {
          host: this.hostName,
          uri: this.requestUri,
          pageNumber: this.pageNumber - 1,
          pageSize: this.pageSize
        }

        let postresult = await this.axios.post(uri, requestBody)

        console.log(postresult.data.data)
        if (postresult.data.data != null) {
          this.mockRulesTotalSize = postresult.data.data.totalElements
          this.data = postresult.data.data.content
        } else {
          this.data = []
        }

        this.$refs.noticeinformation.clear()
    },

    addMockRule: async function() {
      this.addRule.id = null
      this.addRule.update = false
      this.addRule.uri = null
      this.addRule.mockResponse = null
      this.addRule.responseHeaders = null
      this.modalTitle = "添加Mock规则"
      this.addRuleModal = true
    },
    deleteMockRule: async function(params) {
      this.addRule.host = params.row.host
      this.addRule.uri = params.row.uri
      this.addRule.mockResponse = params.row.mockResponse
      this.addRule.id = params.row.id
      this.addRule.update = true
      this.deleteRuleModal = true
      if(params.row.workMode == 'UPSTREAM'){
        this.addRule.upstreams = params.row.upstreams
        this.addRule.workMode = 'UPSTREAM'
        this.showUpstreamMode = true
      }

    },
    updateMockRule: async function(params) {
      this.addRule.host = params.row.host
      this.addRule.uri = params.row.uri
      this.addRule.mockResponse = params.row.mockResponse
      this.addRule.id = params.row.id
      this.addRule.responseHeaders =
        params.row.responseHeaders == null
          ? null
          : JSON.stringify(params.row.responseHeaders)
      this.addRule.update = true
      this.modalTitle = "修改Mock规则"
      this.addRuleModal = true

      if(params.row.workMode == 'UPSTREAM'){
        this.addRule.upstreams = params.row.upstreams
        this.addRule.workMode = 'UPSTREAM'
        this.showUpstreamMode = true
      }

    },
    copyMockRule: async function(params) {
      this.addRule.host = params.row.host
      this.addRule.id = null
      this.addRule.uri = params.row.uri
      this.addRule.mockResponse = params.row.mockResponse
      this.addRule.responseHeaders =
        params.row.responseHeaders == null
          ? null
          : JSON.stringify(params.row.responseHeaders)
      this.addRule.update = false
      this.modalTitle = "根据已有规则创建Mock规则"
      this.addRuleModal = true

      if(params.row.workMode == 'UPSTREAM'){
        this.addRule.upstreams = params.row.upstreams
        this.addRule.workMode = 'UPSTREAM'
        this.showUpstreamMode = true
      }

    },
    addOk: async function() {
      let uri

      if (!this.addRule.update)
        uri = this.server + "/api/mock/2.0/addRule" + ""
      else uri = this.server + "/api/mock/2.0/updateRule" + ""

      //let requestBody = {'hostName':this.hostName,'uri':this.requestUri}

      let postBody = lodash.cloneDeep(this.addRule)

      postBody.responseHeaders = JSON.parse(postBody.responseHeaders)

      let postresult = await this.axios.post(uri, postBody)

      console.log(postresult.data.data)

      if (postresult.data.success) {
        await this.queryMockRules()
        this.$refs.noticeinformation.showalert("success", "添加/修改成功")
      } else {
        this.$refs.noticeinformation.showalert(
          "error",
          "添加/修改失败:" + postresult.data.message
        )
      }
    },
    deleteOk: async function() {
      let uri = this.server + "/api/mock/2.0/deleteRule" + ""
      let postresult = await this.axios.post(uri, { id: this.addRule.id })
      if (postresult.data.success) {
        await this.queryMockRules()
        this.$refs.noticeinformation.showalert("success", "删除成功")
      } else {
        this.$refs.noticeinformation.showalert(
          "error",
          "删除失败:" + postresult.data.message
        )
      }
    },
    cancel: async function() {}
  }
}
</script>
<style scoped>
.modalInputLabel {
  width: 110px;
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
/* 
.nextedForm div{
  margin-right:5px
  
} */
</style>

<style>

.ivu-tooltip-inner{
  word-break:break-all;
  overflow: auto;
  max-height: 600px;
}
</style>
