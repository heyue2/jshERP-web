<template>
  <j-modal
    :title="title"
    :width="width"
    :visible="visible"
    :confirmLoading="confirmLoading"
    :maskClosable="false"
    :keyboard="false"
    :forceRender="true"
    v-bind:prefixNo="prefixNo"
    switchHelp
    switchFullscreen
    @ok="handleOk"
    @cancel="handleCancel"
    wrapClassName="ant-modal-cust-warp"
    :id="prefixNo"
    style="top:5%;height: 100%;overflow-y: hidden">
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-row class="form-row" :gutter="24">

<!--          <a-col :lg="6" :md="12" :sm="24">-->
<!--            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="供应商" data-step="1" data-title="供应商"-->
<!--              data-intro="供应商必须选择，如果发现需要选择的供应商尚未录入，可以在下拉框中点击新增供应商进行录入">-->
<!--              <a-select placeholder="选择供应商" v-decorator="[ 'organId', validatorRules.organId ]"-->
<!--                :dropdownMatchSelectWidth="false" showSearch optionFilterProp="children">-->
<!--                <div slot="dropdownRender" slot-scope="menu">-->
<!--                  <v-nodes :vnodes="menu" />-->
<!--                  <a-divider style="margin: 4px 0;" />-->
<!--                  <div v-if="isTenant" style="padding: 4px 8px; cursor: pointer;"-->
<!--                       @mousedown="e => e.preventDefault()" @click="addSupplier"><a-icon type="plus" /> 新增供应商</div>-->
<!--                </div>-->
<!--                <a-select-option v-for="(item,index) in supList" :key="index" :value="item.id">-->
<!--                  {{ item.supplier }}-->
<!--                </a-select-option>-->
<!--              </a-select>-->
<!--            </a-form-item>-->
<!--          </a-col>-->

          <a-col :lg="6" :md="12" :sm="24">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="客户" data-step="1" data-title="客户"
                         data-intro="客户必须选择，如果发现需要选择的客户尚未录入，可以在下拉框中点击新增客户进行录入。
                          特别注意，客户如果录入之后在下拉框中不显示，请检查是否给当前用户分配对应的客户权限">
              <a-select placeholder="选择客户" v-decorator="[ 'organId', validatorRules.organId ]"
                        :dropdownMatchSelectWidth="false" showSearch optionFilterProp="children">
                <div slot="dropdownRender" slot-scope="menu">
                  <v-nodes :vnodes="menu" />
                  <a-divider style="margin: 4px 0;" />
                  <div v-if="isTenant" style="padding: 4px 8px; cursor: pointer;"
                       @mousedown="e => e.preventDefault()" @click="addCustomer"><a-icon type="plus" /> 新增客户</div>
                </div>
                <a-select-option v-for="(item,index) in cusList" :key="index" :value="item.id">
                  {{ item.supplier }}
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>

          <a-col :lg="6" :md="12" :sm="24">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="单据名称"   data-step="2"  data-title="单据名称" data-intro="单据的名称 方便自己来区分报价单">
              <a-input placeholder="请输入单据名称" v-decorator.trim="[ 'name' ]" />
            </a-form-item>
          </a-col>

          <a-col :lg="6" :md="12" :sm="24">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="单据日期">
              <j-date v-decorator="['operTime', validatorRules.operTime]" :show-time="true"/>
            </a-form-item>
          </a-col>
          <a-col :lg="6" :md="12" :sm="24">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="单据编号" data-step="3" data-title="单据编号"
              data-intro="单据编号自动生成、自动累加、开头是单据类型的首字母缩写，累加的规则是每次打开页面会自动占用一个新的编号">
              <a-input placeholder="请输入单据编号" v-decorator.trim="[ 'number' ]" :readOnly="true"/>
            </a-form-item>
          </a-col>
          <a-col :lg="6" :md="12" :sm="24"></a-col>
        </a-row>
        <j-editable-table id="billModal"
          :ref="refKeys[0]"
          :loading="materialTable.loading"
          :columns="materialTable.columns"
          :dataSource="materialTable.dataSource"
          :maxHeight="300"
          :rowNumber="false"
          :rowSelection="true"
          :actionButton="true"
          :dragSort="true"
          @valueChange="onValueChange"
          @added="onAdded"
          @deleted="onDeleted">
          <template #buttonAfter>
            <a-row :gutter="24" style="float:left;" data-step="4" data-title="扫码录入" data-intro="此功能支持扫码枪扫描商品条码进行录入">
              <a-col v-if="scanStatus" :md="6" :sm="24">
                <a-button @click="scanEnter">扫码录入</a-button>
              </a-col>
              <a-col v-if="!scanStatus" :md="16" :sm="24" style="padding: 0 6px 0 12px">
                <a-input placeholder="请扫码商品条码并回车" v-model="scanBarCode" @pressEnter="scanPressEnter" ref="scanBarCode"/>
              </a-col>
              <a-col v-if="!scanStatus" :md="6" :sm="24" style="padding: 0px">
                <a-button @click="stopScan">收起扫码</a-button>
              </a-col>
            </a-row>
          </template>
        </j-editable-table>
        <a-row class="form-row" :gutter="24">
          <a-col :lg="24" :md="24" :sm="24">
            <a-form-item :labelCol="labelCol" :wrapperCol="{xs: { span: 24 },sm: { span: 24 }}" label="">
              <a-textarea :rows="1" placeholder="请输入备注" v-decorator="[ 'remark' ]" style="margin-top:8px;"/>
            </a-form-item>
          </a-col>
        </a-row>
        <a-row class="form-row" :gutter="24">
          <a-col :lg="6" :md="12" :sm="24">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="优惠率" data-step="6" data-title="优惠率"
                         data-intro="针对单据明细中商品总金额进行优惠的比例">
              <a-input style="width:185px;" placeholder="请输入优惠率" v-decorator.trim="[ 'discount' ]" suffix="%" @keyup="onKeyUpDiscount"/>
            </a-form-item>
          </a-col>
          <a-col :lg="6" :md="12" :sm="24">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="付款优惠" data-step="7" data-title="付款优惠"
                         data-intro="针对单据明细中商品总金额进行优惠的金额">
              <a-input placeholder="请输入付款优惠" v-decorator.trim="[ 'discountMoney' ]" @keyup="onKeyUpDiscountMoney"/>
            </a-form-item>
          </a-col>
          <a-col :lg="6" :md="12" :sm="24">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="优惠后金额" data-step="8" data-title="优惠后金额"
                         data-intro="针对单据明细中商品总金额进行优惠后的金额">
              <a-input placeholder="请输入优惠后金额" v-decorator.trim="[ 'discountLastMoney' ]" :readOnly="true"/>
            </a-form-item>
          </a-col>
          <a-col :lg="6" :md="12" :sm="24">
          </a-col>
        </a-row>
        <a-row class="form-row" :gutter="24">
          <a-col :lg="6" :md="12" :sm="24">
            <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="附件" data-step="5" data-title="附件" data-intro="可以上传与单据相关的图片、文档，支持多个文件">
              <j-upload v-model="fileList" bizPath="bill"></j-upload>
            </a-form-item>
          </a-col>
        </a-row>
      </a-form>
    </a-spin>
    <customer-modal ref="customerModalForm" @ok="customerModalFormOk"></customer-modal>
    <vendor-modal ref="vendorModalForm" @ok="vendorModalFormOk"></vendor-modal>
  </j-modal>
</template>
<script>
  import pick from 'lodash.pick'
  import VendorModal from '../../system/modules/VendorModal'
  import CustomerModal from '../../system/modules/CustomerModal'
  import { FormTypes } from '@/utils/JEditableTableUtil'
  import { JEditableTableMixin } from '@/mixins/JEditableTableMixin'
  import { BillModalMixin } from '../mixins/BillModalMixin'
  import { getMpListShort,handleIntroJs } from "@/utils/util"
  import JUpload from '@/components/jeecg/JUpload'
  import JDate from '@/components/jeecg/JDate'
  import Vue from 'vue'
  export default {
    name: "PurchaseOrderModal",
    mixins: [JEditableTableMixin,BillModalMixin],
    components: {
      CustomerModal,
      VendorModal,
      JUpload,
      JDate,
      VNodes: {
        functional: true,
        render: (h, ctx) => ctx.props.vnodes,
      }
    },
    data () {
      return {
        title:"操作",
        width: '1600px',
        moreStatus: false,
        // 新增时子表默认添加几行空数据
        addDefaultRowNum: 1,
        visible: false,
        supList: [],
        depotList: [],
        operTimeStr: '',
        prefixNo: 'CGDD',
        fileList:[],
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 8 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },
        refKeys: ['materialDataTable', ],
        tableKeys:['materialDataTable', ],
        activeKey: 'materialDataTable',
        materialTable: {
          loading: false,
          dataSource: [],
          columns: [
            // { title: '仓库名称', key: 'depotId', width: '7%', type: FormTypes.hidden },
            { title: '条码', key: 'barCode', width: '8%', type: FormTypes.popupJsh, kind: 'material', multi: true,
              validateRules: [{ required: true, message: '${title}不能为空' }]
            },
            { title: '名称', key: 'name', width: '6%', type: FormTypes.normal },
            { title: '品牌', key: 'brand', width: '6%', type: FormTypes.normal },
            { title: '规格', key: 'standard', width: '5%', type: FormTypes.normal },
            { title: '型号', key: 'model', width: '5%', type: FormTypes.normal },
            { title: '颜色', key: 'color', width: '5%', type: FormTypes.normal },
            { title: '扩展信息', key: 'materialOther', width: '5%', type: FormTypes.normal },
           // { title: '库存', key: 'stock', width: '5%', type: FormTypes.normal },
            { title: '供应商', key: 'supplierName', width: '5%', type: FormTypes.normal },
            { title: '', key: 'supplierId', width: '0%', type: FormTypes.normal},
            { title: '单位', key: 'unit', width: '4%', type: FormTypes.normal },
            { title: '多属性', key: 'sku', width: '4%', type: FormTypes.normal },
            { title: '数量', key: 'operNumber', width: '5%', type: FormTypes.inputNumber, statistics: true,
              validateRules: [{ required: true, message: '${title}不能为空' }]
            },
            { title: '报价类型', key: 'purchaseType', width: '5%', type: FormTypes.select ,
              options: [{"text":"集采","value":"batchPurchase","selected":true},{"text":"代发","value":"dropshipping"}] },
            { title: '单价', key: 'unitPrice', width: '5%', type: FormTypes.inputNumber ,readonly: true},
            { title: '金额', key: 'allPrice', width: '5%', type: FormTypes.inputNumber, statistics: true },
            { title: '税率', key: 'taxRate', width: '3%', type: FormTypes.inputNumber,placeholder: '%'},
            { title: '税额', key: 'taxMoney', width: '5%', type: FormTypes.inputNumber, readonly: true, statistics: true },
            { title: '价税合计', key: 'taxLastMoney', width: '5%', type: FormTypes.inputNumber, statistics: true },
            { title: '备注', key: 'remark', width: '5%', type: FormTypes.input}
          ]
        },
        confirmLoading: false,
        validatorRules:{
          operTime:{
            rules: [
              { required: true, message: '请输入单据日期!' }
            ]
          },
          organId:{
            rules: [
              { required: true, message: '请选择供应商!' }
            ]
          }
        },
        url: {
          add: '/depotHead/addDepotHeadAndDetail',
          edit: '/depotHead/updateDepotHeadAndDetail',
          detailList: '/depotItem/getDetailList'
        }
      }
    },
    created () {
    },
    methods: {
      //调用完edit()方法之后会自动调用此方法
      editAfter() {
        this.changeColumnHide()
        if (this.action === 'add') {
          this.addInit(this.prefixNo)
          this.fileList = []
          this.$nextTick(() => {
            handleIntroJs(this.prefixNo, 1)
          })
        } else {
          this.model.operTime = this.model.operTimeStr
          this.fileList = this.model.fileName
          this.$nextTick(() => {
            this.form.setFieldsValue(pick(this.model,'organId', 'operTime', 'number', 'remark','name',
            'discount','discountMoney','discountLastMoney'))
          });
          // 加载子表数据
          let params = {
            headerId: this.model.id,
            mpList: getMpListShort(Vue.ls.get('materialPropertyList'))  //扩展属性
          }
          let url = this.readOnly ? this.url.detailList : this.url.detailList;
          this.requestSubTableData(url, params, this.materialTable);
        }
        //复制新增单据-初始化单号和日期
        if(this.action === 'copyAdd') {
          this.model.id = ''
          this.model.tenantId = ''
          this.copyAddInit(this.prefixNo)
        }
        this.initSupplier()
        this.initCustomer()
      },
      /** 整理成formData */
      classifyIntoFormData(allValues) {
        let totalPrice = 0
        let billMain = Object.assign(this.model, allValues.formValue)
        let detailArr = allValues.tablesValue[0].values
        billMain.type = '其它'
        billMain.subType = '采购订单'
        billMain.defaultNumber = billMain.number
        for(let item of detailArr){
          item.depotId = '' //订单不需要仓库
          totalPrice += item.allPrice-0
        }
        billMain.totalPrice = 0-totalPrice
        if(this.fileList && this.fileList.length > 0) {
          billMain.fileName = this.fileList
        }
        if(this.model.id){
          billMain.id = this.model.id
        }
        return {
          info: JSON.stringify(billMain),
          rows: JSON.stringify(detailArr),
        }
      }
    }
  }
</script>
<style scoped>

</style>