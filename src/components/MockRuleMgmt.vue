<template>
  <div>
    <Divider>Http Mock Rule 管理</Divider>
    <Input
      class="input"
      v-model="category"
      placeholder="分组"
      style="width: 300px"
    />
    <Input
      class="input"
      v-model="hostName"
      placeholder="hostName: 例如*"
      style="width: 300px"
    />
    <Input
      class="input"
      v-model="requestUri"
      placeholder="uri: 支持模糊匹配"
      style="width: 300px"
    />
    <Button class="button" type="primary" @click="changePageNumber(1)"
      >查询</Button
    >
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
      :page-size-opts="[10, 20, 30]"
      :page-size="10"
      @on-change="changePageNumber($event)"
      @on-page-size-change="changePageSize($event)"
    />

    <Modal
      width="400px"
      :mask-closable="false"
      v-model="testMockRuleModal"
      title="测试结果"
    >
      <div style="text-align: left; line-break: anywhere">
        {{ testMockRuleResponse }}
      </div>
      <div slot="footer"></div>
    </Modal>

    <Modal
      width="1024px"
      :mask-closable="false"
      v-model="querylogModal"
      title="请求日志"
    >
      <div style="text-align: left; line-break: anywhere">
        <Table
          border
          :columns="requestlogcolumns"
          :data="requestlogdata"
        ></Table>
        <Page
          class="page"
          :total="requestlogTotalSize"
          :current="requestlogPageNumber"
          show-total
          show-sizer
          :page-size-opts="[7, 10, 15]"
          :page-size="7"
          @on-change="changeRequestLogPageNumber($event)"
          @on-page-size-change="changeRequestLogPageSize($event)"
        />
      </div>
      <div slot="footer"></div>
    </Modal>

    <!-- start of the add modal -->
    <Modal
      width="600px"
      :mask-closable="false"
      v-model="addRuleModal"
      :title="modalTitle"
      @on-ok="addOk"
      @on-cancel="cancel"
    >
      <div>
        <span class="modalInputLabel">分组:</span>
        <Input
          v-model="addRule.category"
          placeholder="mock规则的分组"
          style="width: 400px"
        />
      </div>

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
        <Select
          v-model="addRule.workMode"
          style="width: 200px"
          @on-change="changeWorkMode($event)"
        >
          <Option
            v-for="item in workModeOption"
            :value="item.value"
            :key="item.value"
            >{{ item.label }}</Option
          >
        </Select>
      </div>

      <div>
        <span class="modalInputLabel">请求Uri:</span>
        <Input
          v-model="addRule.uri"
          placeholder="要匹配的Uri路径,不含协议地址和端口号,以/开头"
          style="width: 400px"
        />
      </div>
      <div v-if="showUpstreamMode">
        <div class="nextedForm">
          <span class="modalInputLabel">上游服务: </span>

          <Input
            v-model="addRule.upstreams.nodes[0].protocol"
            placeholder="http"
            style="width: 50px"
          />
          ://
          <Input
            v-model="addRule.upstreams.nodes[0].address"
            placeholder="localhost:8081"
            style="width: 150px"
          />
          <Input
            v-model="addRule.upstreams.nodes[0].uri"
            placeholder="/testuri"
            style="width: 150px"
          />
        </div>
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

      <div v-if="!showUpstreamMode">
        <span class="modalInputLabel">响应Mock报文:</span>
        <Input
          v-model="addRule.mockResponse"
          type="textarea"
          :rows="10"
          placeholder="mock response body"
        />
      </div>

      <Button class="testMockRule" type="info" @click="testMockRule()"
        >测试响应报文</Button
      >
    </Modal>

    <!-- end of the add modal -->

    <!-- start of the delete modal -->
    <Modal
      width="600px"
      :mask-closable="false"
      v-model="deleteRuleModal"
      title="删除Mock规则"
      @on-ok="deleteOk"
      @on-cancel="cancel"
    >
      <div>
        <span class="modalInputLabel">请求Host:</span>
        <Input
          disabled
          v-model="addRule.host"
          placeholder="要添加的HostName,为空或者* 则表示会匹配所有HostName"
          style="width: 400px"
        />
      </div>
      <div>
        <span class="modalInputLabel">工作模式:</span>
        <Select
          disabled
          v-model="addRule.workMode"
          style="width: 200px"
          @on-change="changeWorkMode($event)"
        >
          <Option
            v-for="item in workModeOption"
            :value="item.value"
            :key="item.value"
            >{{ item.label }}</Option
          >
        </Select>
        <div v-if="showUpstreamMode">
          <div class="nextedForm">
            <span class="modalInputLabel">上游服务: </span>

            <Input
              disabled
              v-model="addRule.upstreams.nodes[0].protocol"
              placeholder="http"
              style="width: 50px"
            />
            ://
            <Input
              disabled
              v-model="addRule.upstreams.nodes[0].address"
              placeholder="localhost:8081"
              style="width: 150px"
            />
            <Input
              disabled
              v-model="addRule.upstreams.nodes[0].uri"
              placeholder="/testuri"
              style="width: 150px"
            />
          </div>
        </div>
      </div>

      <div>
        <span class="modalInputLabel">请求Uri:</span>
        <Input disabled v-model="addRule.uri" style="width: 400px" />
      </div>

      <div v-if="!showUpstreamMode">
        <span class="modalInputLabel">响应Headers:</span>
        <Input
          disabled
          v-model="addRule.responseHeaders"
          type="textarea"
          :rows="3"
        />
      </div>

      <div v-if="!showUpstreamMode">
        <span class="modalInputLabel">响应Mock报文:</span>
        <Input
          disabled
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

import lodash from "lodash";

export default {
  created: function () {
    this.queryMockRules();
  },
  data() {
    return {
      testMockRuleModal: false,
      querylogModal: false,
      querylogByHostName: "",
      querylogByUri: "",
      testMockRuleResponse: "",
      server: this.$store.getters.getServer,
      hostName: "",
      category: "",
      requestUri: null,
      mockRulesTotalSize: 0,
      requestlogTotalSize: 0,
      pageSize: 10,
      pageNumber: 1,
      requestlogPageSize: 7,
      requestlogPageNumber: 1,
      modalTitle: "",
      workModeOption: [
        {
          value: "MOCK",
          label: "Mock",
        },
        {
          value: "UPSTREAM",
          label: "Upstream",
        },
      ],
      showUpstreamMode: false,
      addRule: {
        enable: true,
        id: null,
        host: "*",
        uri: null,
        mockResponse: null,
        workMode: "MOCK",
        upstreams: {
          nodes: [
            {
              protocol: "",
              address: "",
              uri: "",
            },
          ],
        },
        update: false,
        responseHeaders: null,
        category: null,
      },
      update: null,
      addRuleModal: false,
      deleteRuleModal: false,
      requestlogdata: [],
      requestlogcolumns: [
        {
          title: "请求时间",
          width: 180,
          key: "createTime",
          render: (h, params) => {
            let date = new Date(parseInt(params.row.createTime));
            //this.$log.debug(params)
            return h(
              "span",
              {},
              date.toLocaleDateString() + date.toLocaleTimeString()
            );
          },
        },
        {
          title: "Mock",
          key: "mock",
          width: 100,
        },
        {
          title: "实际请求uri",
          key: "requestUri",
        },
        {
          title: "请求头",
          key: "requestHeaders",
          render: this.renderRequestLogRequestHeaders,
        },
        {
          title: "请求报文",
          key: "requestBody",
          render: this.renderRequestLogRequestBody,
        },
        {
          title: "响应头",
          key: "responseHeaders",
          render: this.renderRequestLogResponseHeaders,
        },
        {
          title: "响应报文",
          key: "responseBody",
          render: this.renderRequestLogResponseBody,
        },
      ],
      columns: [
        /*
                {
                  title: "id",
                  key: "id",
                  width:120
                  // render: (h, params) => {
                  //   return h("div", params.row._id.$oid)
                  // }
                },
                */
        {
          title: "分组",
          key: "category",
          width: 100,
        },
        {
          title: "hostName",
          key: "host",
          width: 200,
        },
        {
          title: "uri",
          key: "uri",
          width: 250,
        },
        {
          title: "工作模式",
          width: 100,
          key: "workMode",
        },
        {
          title: "上游节点",
          render: this.renderUpstreamNodes,
          width: 180,
        },
        {
          title: "响应Headers",
          width: 150,
          render: this.renderMockResponseHeaders,
        },
        {
          title: "响应mock报文",
          key: "mockResponse",
          render: this.renderMockResponseColumn,
        },
        {
          title: "操作",
          key: "action",
          width: 200,
          align: "center",
          render: this.renderActionColumn,
        },
      ],
      data: [],
      newaxios: this.axios.create({
        withCredentials: true,
      }),
    };
  },
  methods: {
    renderUpstreamNodes: function (h, params) {
      let upstreamNodes = "N/A";
      if (params.row.workMode == "UPSTREAM") {
        upstreamNodes = params.row.upstreams.nodes;
      }

      return h("div", this.showless(upstreamNodes));
    },

    renderMockResponseColumn: function (h, params) {
      let mockResponse = "N/A";
      if (params.row.workMode == "MOCK") {
        mockResponse =
          params.row.mockResponse == null ? "无" : params.row.mockResponse;
        return h(
          "Tooltip",
          {
            props: {
              transfer: true,
              theme: "light",
              placement: "left-start",
              "max-width": "500",
              content: mockResponse,
            },
          },
          [h("div", this.showless(mockResponse))]
        );
      } else {
        return h("div", mockResponse);
      }
    },
    renderRequestLogRequestHeaders: function (h, params) {
      return this.renderToolTipShowless(h, params.row.requestHeaders);
    },
    renderRequestLogResponseHeaders: function (h, params) {
      return this.renderToolTipShowless(h, params.row.responseHeaders);
    },
    renderRequestLogRequestBody: function (h, params) {
      return this.renderToolTipShowless(h, params.row.requestBody);
    },
    renderRequestLogResponseBody: function (h, params) {
      return this.renderToolTipShowless(h, params.row.responseBody);
    },
    renderToolTipShowless: function (h, content) {
      let contentToString =
        typeof content === "object" && content !== null
          ? JSON.stringify(content)
          : content;

      return h(
        "Tooltip",
        {
          props: {
            transfer: true,
            theme: "light",
            placement: "left-start",
            "max-width": "500",
            content: contentToString,
          },
        },
        [h("div", this.showless(contentToString))]
      );
    },
    renderMockResponseHeaders: function (h, params) {
      let mockResponseHeaders = "N/A";
      if (params.row.workMode == "MOCK") {
        mockResponseHeaders =
          params.row.responseHeaders == null
            ? "无"
            : params.row.responseHeaders;
      }
      return h("div", this.showless(mockResponseHeaders));
    },
    renderActionColumn: function (h, params) {
      return h("div", [
        h(
          "Button",
          {
            props: {
              type: "primary",
              size: "small",
            },
            style: {
              margin: "5px",
            },
            on: {
              click: () => {
                this.updateMockRule(params);
              },
            },
          },
          "修改"
        ),
        h(
          "Button",
          {
            props: {
              type: "primary",
              size: "small",
            },
            style: {
              margin: "5px",
            },
            on: {
              click: () => {
                this.copyMockRule(params);
              },
            },
          },
          "复制创建"
        ),
        h(
          "Button",
          {
            props: {
              type: "primary",
              size: "small",
            },
            style: {
              margin: "5px",
            },
            on: {
              click: () => {
                this.queryRequestlogs(params.row.uri, params.row.host);
              },
            },
          },
          "请求记录"
        ),
        h(
          "Button",
          {
            props: {
              type: "error",
              size: "small",
            },
            style: {
              margin: "5px",
            },
            on: {
              click: () => {
                this.deleteMockRule(params);
              },
            },
          },
          "删除"
        ),
      ]);
    },

    changeWorkMode: function (workMode) {
      if (workMode == "UPSTREAM") {
        this.showUpstreamMode = true;
        this.responseBody = false;
        this.responseHeaders = false;
      } else {
        this.showUpstreamMode = false;
        this.addRule.upstreams = {
          nodes: [
            {
              protocol: "",
              address: "",
              uri: "",
            },
          ],
        };
        this.responseBody = true;
        this.responseHeaders = true;
      }
    },

    showless: function (message) {
      let showlessMessage =
        typeof message === "string" || message instanceof String
          ? message
          : JSON.stringify(message);

      return showlessMessage.length > 128
        ? showlessMessage.substring(0, 128) + " ...更多内容"
        : showlessMessage;
    },
    changeRequestLogPageSize: async function (size) {
      this.requestlogPageSize = size;
      this.queryRequestlogs(this.querylogByUri, this.querylogByHostName);
    },
    changeRequestLogPageNumber: async function (number) {
      this.$log.debug("------- " + number);
      this.requestlogPageNumber = number;
      this.queryRequestlogs(this.querylogByUri, this.querylogByHostName);
    },
        queryRequestLogfirstPage:async function(mockRuleUri, mockRuleHostName){
	this.requestlogdata = [];
       this.requestlogPageNumber = 1;
       this.requestlogPageSize = 7;
       this.queryRequestlogs(mockRuleUri, mockRuleHostName);
    },

    changePageSize: async function (size) {
      this.pageSize = size;
      this.queryMockRules();
    },
    changePageNumber: async function (number) {
      this.pageNumber = number;
      this.queryMockRules();
    },
    queryMockRules: async function () {
      this.$log.debug(this);
      let uri = this.server + "/api/mock/2.0/queryRule" + "";

      let requestBody = {
        category: this.category,
        host: this.hostName,
        uri: this.requestUri,
        pageNumber: this.pageNumber - 1,
        pageSize: this.pageSize,
      };

      let postresult = await this.newaxios.post(uri, requestBody);

      this.$log.debug(postresult.data.data);
      if (postresult.data.data != null) {
        this.mockRulesTotalSize = postresult.data.data.totalElements;
        this.data = postresult.data.data.content;
      } else {
        this.data = [];
      }

      this.$refs.noticeinformation.clear();
    },
    queryRequestlogs: async function (mockRuleRri, mockRuleHostName) {
      this.querylogByHostName = mockRuleHostName;
      this.querylogByUri = mockRuleRri;
      this.querylogModal = true;

      let uri = this.server + "/api/mock/2.0/queryRequestLog" + "";

      let requestBody = {
        uri: mockRuleRri,
        hostname: mockRuleHostName,
        pageNumber: this.requestlogPageNumber - 1,
        pageSize: this.requestlogPageSize,
      };

      let postresult = await this.newaxios.post(uri, requestBody);

      this.$log.debug(postresult);
      if (postresult.data.data != null && postresult.data.data != undefined) {
        this.requestlogTotalSize = postresult.data.data.totalElements;
        this.requestlogdata = postresult.data.data.content;
      } else 
      { this.requestlogdata = []; this.requestlogTotalSize = 0; }
    },
    emptyMockRuleData: function () {
      this.addRule.id = null;
      this.addRule.update = false;
      this.addRule.uri = null;
      this.addRule.mockResponse = null;
      this.addRule.responseHeaders = null;
      this.addRule.category = null;
    },
    assignMockRuleData: function (params) {
      this.addRule.host = params.row.host;
      this.addRule.uri = params.row.uri;
      this.addRule.mockResponse = params.row.mockResponse;
      this.addRule.id = params.row.id;
      this.addRule.update = true;
      this.addRule.category = params.row.category;
      if (params.row.workMode == "UPSTREAM") {
        this.addRule.upstreams = params.row.upstreams;
        this.addRule.workMode = "UPSTREAM";
      } else {
        this.addRule.workMode = "MOCK";
      }
      this.addRule.responseHeaders =
        params.row.responseHeaders == null
          ? null
          : JSON.stringify(params.row.responseHeaders);
    },
    addMockRule: async function () {
      this.emptyMockRuleData();
      this.modalTitle = "添加Mock规则";
      this.addRuleModal = true;
    },
    deleteMockRule: async function (params) {
      this.assignMockRuleData(params);

      if (params.row.workMode == "UPSTREAM") {
        this.showUpstreamMode = true;
      }

      this.deleteRuleModal = true;
    },
    updateMockRule: async function (params) {
      this.assignMockRuleData(params);

      this.modalTitle = "修改Mock规则";
      this.addRuleModal = true;

      if (params.row.workMode == "UPSTREAM") {
        this.showUpstreamMode = true;
      } else {
        this.showUpstreamMode = false;
      }
    },
    copyMockRule: async function (params) {
      this.assignMockRuleData(params);
      this.addRule.id = null;

      this.modalTitle = "根据已有规则创建Mock规则";

      if (params.row.workMode == "UPSTREAM") {
        this.showUpstreamMode = true;
      } else {
        this.showUpstreamMode = false;
      }
      this.addRule.update = false;
      this.modalTitle = "复制Mock规则";
      this.addRuleModal = true;
    },
    addOk: async function () {
      let uri;

      if (!this.addRule.update)
        uri = this.server + "/api/mock/2.0/addRule" + "";
      else uri = this.server + "/api/mock/2.0/updateRule" + "";

      //let requestBody = {'hostName':this.hostName,'uri':this.requestUri}

      let postBody = lodash.cloneDeep(this.addRule);

      postBody.responseHeaders = JSON.parse(postBody.responseHeaders);

      let postresult = await this.newaxios.post(uri, postBody);

      this.$log.debug(postresult.data.data);

      if (postresult.data.success) {
        await this.queryMockRules();
        this.$refs.noticeinformation.showalert("success", "添加/修改成功");
      } else {
        this.$refs.noticeinformation.showalert(
          "error",
          "添加/修改失败:" + postresult.data.message
        );
      }
    },
    deleteOk: async function () {
      let uri = this.server + "/api/mock/2.0/deleteRule" + "";
      let postresult = await this.newaxios.post(uri, {
        id: this.addRule.id,
      });
      if (postresult.data.success) {
        await this.queryMockRules();
        this.$refs.noticeinformation.showalert("success", "删除成功");
      } else {
        this.$refs.noticeinformation.showalert(
          "error",
          "删除失败:" + postresult.data.message
        );
      }
    },
    cancel: async function () {},
    testMockRule: async function () {
      let uri = this.server + "/api/mock/2.0/testRule" + "";
      let postBody = lodash.cloneDeep(this.addRule);

      postBody.responseHeaders = JSON.parse(postBody.responseHeaders); // 将字符串转换为json object

      let postresult = await this.newaxios.post(uri, postBody);

      this.testMockRuleModal = true;
      this.testMockRuleResponse = postresult.data.message;
    },
  },
};
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
.testMockRule {
  margin-top: 10px;
}
</style><style>
.ivu-tooltip-inner {
  word-break: break-all;
  overflow: auto;
  max-height: 600px;
}
</style>
