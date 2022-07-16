<template>
  <div>

    <Modal
      width="500px"
      :mask-closable="false"
      v-model="deleteCondirmModal.show"
      title="清空该mock规则下所有条件规则!？"
      @on-ok="          deleteConditionRulesByMockId(deleteCondirmModal.id
          )"
      @on-cancel=""
    >
    </Modal>

    <!-- 不要直接修改parent 传入的props值，这里为了方便，先这么使用，后续需要优化。  conditionRules.show -->
    <Modal
      width="1024px"
      :mask-closable="false"
      v-model="conditionRules.show"
      title="条件规则管理"
    >
    <span style="color:orange;">备注： 如果未命中任何条件规则，则返回mock规则的默认返回值。若命中条件规则，则返回该条件规则的mock定义。</span>
      <div class="modalElementGroup">
        Mock规则Id:【{{
          conditionRules.params == undefined
            ? ""
            : conditionRules.params.row.id
        }}】 请求Host: 【{{
          conditionRules.params == undefined
            ? ""
            : conditionRules.params.row.host
        }}】 请求Uri: 【{{
          conditionRules.params == undefined
            ? ""
            : conditionRules.params.row.uri
        }}】 下的条件规则列表：
      </div>

      <Button
        class="modalElementGroup"
        type="info"
        @click="
          queryConditionRulesByMockId(
            conditionRules.params == undefined
              ? ''
              : conditionRules.params.row.id
          )
        "
        >查询</Button
      >

      <Button
        class="modalElementGroup"
        type="info"
        @click="openConditionModal()"
        >添加</Button
      >

      <Button
        class="modalElementGroup"
        type="error"
        @click="
            deleteCondirmModal.show = true;
            deleteCondirmModal.id = conditionRules.params == undefined
              ? ''
              : conditionRules.params.row.id ;
        "
        >清空</Button
      >

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
      title="添加或更新条件规则"
      @on-ok="addOk"
      @on-cancel="addCancel"
    >
      <div class="modalElementGroup">
        <span class="modalInputLabel">条件组合:</span>
        <div
          class="modalElementGroup"
          v-for="(
            expression, index
          ) in tobeAddedConditionRule.conditionExpression"
          :key="index"
        >
          <Input
            v-if="!isSpecialConditionExpression(expression.compareCondition)"
            v-model="expression.toBeCompareValue"
            style="width: 255px"
          />
          <Select v-model="expression.compareCondition"  style="width: 145px">
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
            style="width: 255px"
          />

          <Button type="info" @click="addOrDeleteCondition(index)">{{
            index == 0 ? "+" : "-"
          }}</Button>
        </div>
      </div>

      <div class="modalElementGroup">
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

      <div
        class="modalElementGroup"
        v-if="tobeAddedConditionRule.workMode != 'MOCK'"
      >
        <div class="nextedForm">
          <span class="modalInputLabel">上游服务: </span> <div style="color:orange;"> 【protocol://address/uri】 INTERNAL_FORWARD 内部转发模式下 protocol和address仍为首次请求的地址，仅uri的配置会用于内部转发。</div>

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

      <div
        class="modalElementGroup"
        v-if="tobeAddedConditionRule.workMode == 'MOCK'"
      >
        <span class="modalInputLabel">响应Headers:</span>
        <Input
          v-model="tobeAddedConditionRule.responseHeaders"
          type="textarea"
          :rows="5"
          placeholder="{'header':'value'}"
        />
      </div>

      <div
        class="modalElementGroup"
        v-if="tobeAddedConditionRule.workMode == 'MOCK'"
      >
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
      deleteCondirmModal:{
        show:false,
        deleteId:''},
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
        {
          value: "INTERNAL_FORWARD",
          label: "InternalForward",
        },        
      ],
      conditionExpression: [
        {
          value: "EQUAL",
          label: "==",
        },
        {
          value: "NON_EQUAL",
          label: "!=",
        },
        {
          value: "REGREX_MATCH",
          label: "正则匹配",
        },
        {
          value: "GREATER_THAN",
          label: ">",
        },
        {
          value: "GREATER_OR_EQUAL",
          label: ">=",
        },
        {
          value: "LESS_THAN",
          label: "<",
        },
        {
          value: "LESS_OR_EQUAL",
          label: "<=",
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
          width: 60,
        },

        {
          title: "条件",
          key: "conditionExpression",
          render:this.renderConditionExpression,
        },
        {
          title: "模式",
          key: "workMode",
          width: 80,
        },
        {
          title: "Mock响应头(Upstream下无效)",
          key: "responseHeaders",
         // render: this.renderResponseHeader,
        },

        {
          title: "Mock响应体或Upstream节点",
          render: this.renderMockResponseColumn
        },
        {
          title: "操作",
          render: this.renderActionColumn,
          width:140
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
            compareCondition: "EQUAL",
            conditionValue: "",
          },
        ],
        orderId: null,
        mockResponse: "",
        responseHeaders: null,
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
      httpConditionRuleDetails: {
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
    openConditionModal:function(){
          this.showAddConditionRuleModal = true; 
          this.clearAddConditionFormData();
    },
    renderMockResponseColumn: function (h, params) {

      let mockResponse = '';
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
        let showUpstreamText = '';
        let node = params.row.upstreams.nodes[0];
        showUpstreamText+= node.protocol+'://'+node.address+node.uri+'\r\n';
        return h("div", showUpstreamText);
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
    renderConditionExpression:function(h,params){
      let showtext = '';
      this.$log.debug(params.row.conditionExpression.keys());
      this.$log.debug(typeof(params.row.conditionExpression));
      let thiz = this;
      params.row.conditionExpression.forEach(
          function(element){

            if(thiz.isSpecialConditionExpression(element.compareCondition))
            {
            showtext =  showtext+thiz.getConditionExpressLabel(element.compareCondition)+' ';

            }
            else{
            showtext =  showtext+element.toBeCompareValue+' '+thiz.getConditionExpressLabel(element.compareCondition)+' '+element.conditionValue+' ';
            }
          }

      );

      return h('div',{},showtext);
    },

    getConditionExpressLabel:function(expression){

      let label = '==';
      switch(expression){
        case 'EQUAL': 
          label = '==';         
          break;
        case 'NON_EQUAL': 
          label = '!=';         
          break;
        case 'REGREX_MATCH': 
          label = 'regrex_match';         
          break;
        case 'GREATER_THAN': 
          label = '>';         
          break;
        case 'GREATER_OR_EQUAL': 
          label = '>=';         
          break;
        case 'LESS_THAN': 
          label = '<';         
          break;
        case 'LESS_OR_EQUAL': 
          label = '<=';         
          break;
        case 'LEFT_PARENTHESIS': 
          label = '(';         
          break;
        case 'RIGHT_PARENTHESIS': 
          label = ')';         
          break;
        case 'OR': 
          label = '||';         
          break;
        case 'AND': 
          label = '&&';         
          break;                                                                                                                                              
        default :

      }

      return label;

    },

    renderResponseHeader: function (h, params) {

    },
    renderActionColumn: function (h, params) {
      return h("div", [
        h(
          "Button",
          {
            props: {
              type: "info",
              size: "small",
            },
            style: {
              margin: "5px",
            },
            on: {
              click: () => {
                this.showUpdateConditionRuleModal(params);
              },
            },
          },
          "编辑"
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
  showUpdateConditionRuleModal:function(params){

    this.showAddConditionRuleModal = true;
    this.tobeAddedConditionRule = params.row;

  },
    deleteConditionRule: async function (params) {
      // delete specified condition rule
      this.$log.debug(this.httpConditionRuleDetails);
      this.$log.debug(params);
      let conditionRuleId = this.httpConditionRuleDetails.id;
      let uri =
        this.server +
        "/xxxxhissummerxxxx/api/httpConditionRule/" +
        conditionRuleId +
        "/" +
        (params.row.orderId - 1);
      this.$log.debug(uri);
      let postresult = await this.newaxios.delete(uri);
      this.$log.debug(
        "delete specified condition to condition rules : ",
        postresult
      );
      if (postresult.data.success) {
        this.conditionRuleListData = postresult.data.data.conditionRules;
        this.httpConditionRuleDetails = postresult.data.data;
      } else if (postresult.status != 200 || !postresult.data.success) {
        this.error(
          "delete spcefied conditionRule error: ",
          postresult.data.message
        );
      }
    },
    changeWorkMode: function () {},
    isSpecialConditionExpression: function (expression) {
      let isSpecial = (
        expression == "LEFT_PARENTHESIS" ||
        expression == "RIGHT_PARENTHESIS" ||
        expression == "OR" ||
        expression == "AND"
      );
      return isSpecial;
    },
    addOrDeleteCondition: function (index) {
      if (index == 0) {
        this.tobeAddedConditionRule.conditionExpression.push({
          toBeCompareValue: "",
          compareCondition: "",
          conditionValue: "",
        });
      } else {
        this.tobeAddedConditionRule.conditionExpression.splice(index, 1);
      }
    },
    addOk: async function () {
      this.showAddConditionRuleModal = false;
      this.httpConditionRuleDetails.httpMockRuleId = this.conditionRules.params.row.id;
      let isAddCondition = true; // 是否是添加条件规则
      if (
        this.tobeAddedConditionRule.orderId  != null
      )      
      {
          // this.tobeAddedConditionRule.orderId  规则Id不存在，证明是要添加。
          isAddCondition = false;
      }
      if(this.tobeAddedConditionRule.orderId ==null )
       {  this.tobeAddedConditionRule.orderId =
        this.httpConditionRuleDetails.conditionRules.length + 1;
      this.httpConditionRuleDetails.conditionRules.push(
        this.tobeAddedConditionRule
      );
       }
      try {
        this.tobeAddedConditionRule.responseHeaders = JSON.parse(
          this.tobeAddedConditionRule.responseHeaders
        );
      } catch (e) {
        this.tobeAddedConditionRule.responseHeaders = null;
      }

      this.$log.debug(this.httpConditionRuleDetails);

      if (
        isAddCondition
      ) {
        //添加规则，直接调用添加。 
        if(this.httpConditionRuleDetails.id != null && this.httpConditionRuleDetails.id != undefined)
        {
          await this.addNewHttpConditionRule(this.httpConditionRuleDetails.httpMockRuleId,this.httpConditionRuleDetails);
        }
        else{
          await this.addFirstNewHttpConditionRule(this.httpConditionRuleDetails);
        }

      } else {
        //编辑规则,  todo!!!!!!!
        await this.updateHttpConditionRule(this.httpConditionRuleDetails.id, this.tobeAddedConditionRule.orderId, this.tobeAddedConditionRule);
      }
      this.queryConditionRulesByMockId(
        this.httpConditionRuleDetails.httpMockRuleId
      );
    },
    clearAddConditionFormData() {
      this.tobeAddedConditionRule.orderId = null;
      this.tobeAddedConditionRule.responseHeaders = "";
      this.tobeAddedConditionRule.mockResponse = "";
      this.tobeAddedConditionRule.conditionExpression = [
        {
          toBeCompareValue: "",
          compareCondition: "",
          conditionValue: "",
        },
      ];
      this.tobeAddedConditionRule.upstreams = {
        nodes: [
          {
            protocol: "http",
            address: "",
            uri: "",
            upstreamPolicy: null,
          },
        ],
      };
      this.tobeAddedConditionRule.workMode = "MOCK";
    },
    addCancel: async function () {},
    changePageNumber: async function () {},
    changePageSize: async function () {},
    addFirstNewHttpConditionRule: async function (rule) {
      let uri = this.server + "/xxxxhissummerxxxx/api/httpConditionRule";
      this.$log.debug(this.newaxios);
      let postresult = await this.newaxios.put(uri, rule);
      this.$log.debug("add new rules : ", postresult);
      if (postresult.data.success) {
        this.conditionRuleListData = postresult.data.data.conditionRules;
        this.httpConditionRuleDetails = postresult.data.data;
      } else if (postresult.status != 200 || !postresult.data.success) {
        this.error("add conditionRules error: ", postresult.data.message);
      }
    },
    addNewHttpConditionRule: async function (mockruleId,rule) {
      let uri = this.server + "/xxxxhissummerxxxx/api/httpConditionRule/mockRuleId-"+mockruleId;
      this.$log.debug(this.newaxios);
      let postresult = await this.newaxios.post(uri, rule);
      this.$log.debug("add new rules : ", postresult);
      if (postresult.data.success) {
        this.conditionRuleListData = postresult.data.data.conditionRules;
        this.httpConditionRuleDetails = postresult.data.data;
      } else if (postresult.status != 200 || !postresult.data.success) {
        this.error("add conditionRules error: ", postresult.data.message);
      }
    },    
    updateHttpConditionRule: async function (conditionRuleId, conditionId,condition) {
      let uri = this.server + "/xxxxhissummerxxxx/api/httpConditionRule/"+conditionRuleId+'/'+conditionId;
      this.$log.debug(this.newaxios);
      let postresult = await this.newaxios.post(uri, condition);
      if (postresult.data.success) {
        this.conditionRuleListData = postresult.data.data.conditionRules;
        this.httpConditionRuleDetails = postresult.data.data;
      } else if (postresult.status != 200 || !postresult.data.success) {
        this.error("add conditionRules error: ", postresult.data.message);
      }
    },
    deleteConditionRulesByMockId: async function (mockRuleId) {
      if (mockRuleId == "")
        this.error("mock rule id can not be empty! ", postresult.data.message);

      let uri =
        this.server +
        "/xxxxhissummerxxxx/api/httpConditionRule/mockRuleId-" +
        mockRuleId;
      this.$log.debug(this.newaxios);
      let postresult = await this.newaxios.delete(
        uri,
        this.httpConditionRuleDetails
      );
      this.$log.debug("delete conditionrules by mockid : ", postresult);
      if (postresult.data.success) {
        this.queryConditionRulesByMockId(mockRuleId);
      } else if (postresult.status != 200 || !postresult.data.success) {
        this.conditionRuleListData = [];
        this.httpConditionRuleDetails.conditionRules = [];
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
        this.httpConditionRuleDetails = postresult.data.data;
      } else if (postresult.data.data == null) {
        this.conditionRuleListData = [];
        this.httpConditionRuleDetails.conditionRules = [];
        this.httpConditionRuleDetails.id = null;
      } else if (postresult.status != 200 || !postresult.data.success) {
        this.conditionRuleListData = [];
        this.httpConditionRuleDetails.conditionRules = [];
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
.modalElementGroup {
  margin: 5px;
}
</style>
