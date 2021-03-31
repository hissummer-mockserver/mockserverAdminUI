<template>
  <div>
    <!-- 不要直接修改parent 传入的props值，这里为了方便，先这么使用，后续需要优化。  conditionRules.show -->
    <Modal
      width="800px"
      :mask-closable="false"
      v-model="conditionRules.show"
      title="条件规则管理"
    >
      <div>
        Mock规则【{{
          conditionRules.params == undefined
            ? ""
            : conditionRules.params.row.id
        }}】
        <span class="modalInputLabel">请求Host:</span>
        <span
          >【{{
            conditionRules.params == undefined
              ? ""
              : conditionRules.params.row.host
          }}】</span
        >
        <span class="modalInputLabel">请求Uri:</span>
        <span
          >【{{
            conditionRules.params == undefined
              ? ""
              : conditionRules.params.row.uri
          }}】</span
        >
        下的条件规则列表：
      </div>

      <Button type="info" @click="queryConditionRulesByMockId(conditionRules.params == undefined
            ? ''
            : conditionRules.params.row.id)"
        >查询</Button
      >

      <Button type="info" @click="showAddConditionRuleModal = true"
        >添加</Button
      >

      <Button type="info" @click="deleteConditionRulesByMockId(conditionRules.params == undefined
            ? ''
            : conditionRules.params.row.id)"
        >删除</Button>

      <Table
        border
        :columns="conditionRuleListHeader"
        :data="conditionRuleListData"
      >
      </Table>

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

      <div slot="footer"></div>
    </Modal>

    <!-- start of the add condition rule modal -->
    <Modal
      width="750px"
      :mask-closable="false"
      v-model="showAddConditionRuleModal"
      title="添加条件规则"
      @on-ok="addOk"
      @on-cancel="addCancel"
    >
      <div>
        <span class="modalInputLabel">条件组合:</span>

        <div
          v-for="(
            expression, index
          ) in tobeAddedConditionRule.conditionExpression"
          :key="index"
        >
          <Input
            v-if="!isSpecialConditionExpression(expression.compareCondition)"
            v-model="expression.toBeCompareValue"
            style="width: 50px"
          />
                  <Select
          v-model="expression.compareCondition"
          style="width: 200px"
        >
          <Option
            v-for="item in conditionExpression"
            :value="item.value"
            :key="item.value"
            >{{ item.label }}</Option
          >
        </Select>
          
          <Input
            v-if="!isSpecialConditionExpression(expression.compareCondition)"
            v-model="expression.conditionValue"
            style="width: 50px"
          />
        </div>
      </div>

      <div>
        <span class="modalInputLabel">工作模式:</span>
        <Select
          v-model="tobeAddedConditionRule.workMode"
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

      <div v-if="tobeAddedConditionRule.workMode != 'MOCK'">
        <div class="nextedForm">
          <span class="modalInputLabel">上游服务: </span>

          <Input
            v-model="tobeAddedConditionRule.upstreams.nodes[0].protocol"
            placeholder="http"
            style="width: 50px"
          />
          ://
          <Input
            v-model="tobeAddedConditionRule.upstreams.nodes[0].address"
            placeholder="ip_or_domain:port"
            style="width: 150px"
          />
          <Input
            v-model="tobeAddedConditionRule.upstreams.nodes[0].uri"
            placeholder="/testuri"
            style="width: 150px"
          />
        </div>
      </div>

      <div v-if="tobeAddedConditionRule.workMode == 'MOCK'">
        <span class="modalInputLabel">响应Headers:</span>
        <Input
          v-model="tobeAddedConditionRule.responseHeaders"
          type="textarea"
          :rows="5"
          placeholder="{'header':'value'}"
        />
        <span class="modalInputLabel">响应Mock报文:</span>
        <Input
          v-model="tobeAddedConditionRule.mockResponse"
          type="textarea"
          :rows="10"
          placeholder="mock response body"
        />
      </div>
    </Modal>

    <!-- end of the add condition rule modal -->
  </div>
</template>

<script>
import lodash from "lodash";

export default {
  created: function () {
    this.$log.debug("conditional rule mgmt modal!");
  },

  props: {
    conditionRules: Object,
  },
  data() {
    return {
      mockRulesTotalSize: 0,
      pageNumber: 1,
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
      conditionExpression: [
        {
          value: "EQUAL",
          label: "EQUAL",
        },
        {
          value: "NON_EQUAL",
          label: "NON_EQUAL",
        },
        {
          value: "REGREX_MATCH",
          label: "REGREX_MATCH",
        },
        {
          value: "GREATER_THAN",
          label: "GREATER_THAN",
        },
        {
          value: "GREATER_OR_EQUAL",
          label: "GREATER_OR_EQUAL",
        },
        {
          value: "LESS_THAN",
          label: "LESS_THAN",
        },
        {
          value: "LESS_OR_EQUAL",
          label: "LESS_OR_EQUAL",
        },
        {
          value: "LEFT_PARENTHESIS",
          label: "(",
        },
        {
          value: "RIGHT_PARENTHESIS",
          label: ")",
        },
        {
          value: "OR",
          label: "OR",
        },
        {
          value: "AND",
          label: "AND",
        },
      ],
      showAddConditionRuleModal: false,
      conditionRuleListHeader: [
        {
          title: "序号",
          key: "orderId",
        },

        {
          title: "条件",
          key: "conditionExpression",
        },
        {
          title: "模式",
          key: "workMode",
        },
        {
          title: "自定义响应头",
          key: "responseHeaders",
        },

        {
          title: "响应体",
          key: "mockResponse",
        },
      ],
      conditionRuleListData: [],
      server: this.$store.getters.getServer,
      ifShowModal: false,
      newaxios: this.axios.create({
        withCredentials: true,
      }),
      tobeAddedConditionRule: {
        conditionExpression: [
          {
            toBeCompareValue: "",
            compareCondition: "",
            conditionValue: "",
          },
        ],
        orderId: null,
        mockResponse: "",
        responseHeaders: "",
        upstreams: {
          nodes: [
            {
              protocol: "http",
              address: "",
              uri: "",
              upstreamPolicy: null,
            },
          ],
        },
        workMode: "MOCK",
        enable: true,
      },
      httpConditionRule: {
        //httpconditionrule schema
        id: null,
        httpMockRuleId: null,
        conditionRules: [],
      },
    };
  },
  watch: {
    showModal: function (value, oldvalue) {
      this.$log.debug("changed value: " + value + " " + oldvalue);
    },
    "conditionRules.show": function (value, oldvalue) {
      this.$log.debug("changed value: " + value + " " + oldvalue);
      if (value)
        this.queryConditionRulesByMockId(this.conditionRules.params.row.id);
    },
  },
  methods: {
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
                this.updateConditionRule(params);
              },
            },
          },
          "修改"
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
                this.deleteConditionRule(params);
              },
            },
          },
          "删除"
        ),
      ]);
    },

    changeWorkMode: function () {},
    isSpecialConditionExpression: function (expression) {
      return (
        expression == "(" ||
        expression == ")" ||
        expression == "or" ||
        expression == "and"
      );
    },
    addOk: async function () {
      this.httpConditionRule.httpMockRuleId = this.conditionRules.params.row.id;
      this.tobeAddedConditionRule.orderId =
        this.httpConditionRule.conditionRules.length + 1;
      try {
        this.tobeAddedConditionRule.responseHeaders = JSON.parse(
          this.tobeAddedConditionRule.responseHeaders
        );
      } catch (e) {
        this.tobeAddedConditionRule.responseHeaders = {};
      }
      this.httpConditionRule.conditionRules.push(this.tobeAddedConditionRule);
      this.$log.debug(this.httpConditionRule);

      if (
        this.httpConditionRule.id == null ||
        this.httpConditionRule.id == undefined ||
        this.httpConditionRule.id == null
      ) {
        await this.addNewHttpConditionRule(this.httpConditionRule);
      } else {
        await this.updateHttpConditionRule(this.httpConditionRule);
      }

      this.queryConditionRulesByMockId(this.httpConditionRule.httpMockRuleId);
    },
    addCancel: async function () {},
    changePageNumber: async function () {},
    changePageSize: async function () {},
    updateHttpConditionRule:async function(rule){

      let uri = this.server + "/xxxxhissummerxxxx/api/httpConditionRule/mockRuleId-" +
        rule.httpMockRuleId;
      this.$log.debug(this.newaxios);
      let postresult = await this.newaxios.post(uri, rule);
      this.$log.debug("add new condition to condition rules : ", postresult);
      if (postresult.data.success) {
        this.conditionRuleListData = postresult.data.data.conditionRules;
        this.httpConditionRule = postresult.data.data;
      } else if (postresult.status != 200 || !postresult.data.success) {
        this.error("add conditionRules error: ", postresult.data.message);
      }
    },
    addNewHttpConditionRule: async function (rule) {
      let uri = this.server + "/xxxxhissummerxxxx/api/httpConditionRule";
      this.$log.debug(this.newaxios);
      let postresult = await this.newaxios.put(uri, rule);
      this.$log.debug("add new rules : ", postresult);
      if (postresult.data.success) {
        this.conditionRuleListData = postresult.data.data.conditionRules;
        this.httpConditionRule = postresult.data.data;
      } else if (postresult.status != 200 || !postresult.data.success) {
        this.error("add conditionRules error: ", postresult.data.message);
      }
    },

deleteConditionRulesByMockId:async function(mockRuleId){
      if (mockRuleId == "")
        this.error("mock rule id can not be empty! ", postresult.data.message);

      let uri =
        this.server +
        "/xxxxhissummerxxxx/api/httpConditionRule/mockRuleId-" +
        mockRuleId;
      this.$log.debug(this.newaxios);
      let postresult = await this.newaxios.delete(uri,this.httpConditionRule);
      this.$log.debug("delete conditionrules by mockid : ", postresult);
      if (postresult.data.success) {
          this.queryConditionRulesByMockId(mockRuleId);
      } else if (postresult.status != 200 || !postresult.data.success) {
        this.conditionRuleListData = [];
        this.httpConditionRule.conditionRules = [];
        this.error("delete conditionRules ", postresult.data.message);
      }

},

    queryConditionRulesByMockId: async function (mockRuleId) {
      if (mockRuleId == "")
        this.error("mock rule id can not be empty! ", postresult.data.message);

      let uri =
        this.server +
        "/xxxxhissummerxxxx/api/httpConditionRule/mockRuleId-" +
        mockRuleId;
      this.$log.debug(this.newaxios);
      let postresult = await this.newaxios.get(uri);
      this.$log.debug("query conditionrules by mockid : ", postresult);
      if (postresult.data.data != null) {
        this.conditionRuleListData = postresult.data.data.conditionRules;
        this.httpConditionRule = postresult.data.data;
      } 
       else if (postresult.data.data == null) {
        this.conditionRuleListData = [];
        this.httpConditionRule.conditionRules = [];
        this.httpConditionRule.id = null;
      }
      else if (postresult.status != 200 || !postresult.data.success) {
        this.conditionRuleListData = [];
        this.httpConditionRule.conditionRules = [];
        this.error("query conditionRules ", postresult.data.message);
      }
    },
    error(title, nodesc) {
      this.$Notice.error({
        title: title,
        desc: nodesc,
      });
    },
    info(title, nodesc) {
      this.$Notice.info({
        title: title,
        desc: nodesc,
      });
    },
  },
};
</script>

<style scoped>
</style>
