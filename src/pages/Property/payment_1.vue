<template>
  <div>
    <el-card>
      <div>
        <Comparison
          :type="item.type"
          :key="item.name"
          v-for="item in infoData"
          :data="{name: item.name, value: item.value, chart: item.chart}"></Comparison>
      </div>
    </el-card>
    <el-card>
      <div slot="header">
        <el-select  size="small"
                    v-model="payment_type"
                    clearable
                    @change="fetchListSearch"
                    placeholder="费用类型">
          <el-option
            v-for="item in this.$store.state.dictionary.dictionaryType['payment_type']"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
        <el-input
          placeholder="搜索"
          size="small"
          clearable
          @change="fetchListSearch"
          style="width: 220px; margin-left: 15px"
          prefix-icon="el-icon-search"
          v-model="customer_name">
        </el-input>
        <el-button
          style="float: right"
          type="primary"
          icon="el-icon-plus"
          size="small"
          @click="handleAddContract"
        >新增催缴</el-button>
      </div>
      <GTable
        @row-click="financialState"
        @current-change="handlePageClick"
        @prev-click="handlePageClick"
        @next-click="handlePageClick"
        :page="page"
        :tableLabel="$tableLabels.paymentList"
        :tableData="tableData">
      </GTable>
    </el-card>

    <el-dialog
      title="新建收付款账单"
      v-if="addVisible"
      :visible.sync="addVisible"
      :before-close="(done) => {
         this.$confirm('表单尚未提交确认关闭？')
          .then(_ => {
            done();
          })
          .catch(_ => {});
        }"
      width="600px">
      <div>
        <ParkForm
        @onSubmit="fetchAdd"
        @onCancel="() => {this.addVisible = false}"
        :formList="$formsLabels.paymentForm"
        :options="$store.getters.paymentListOptions"
        :defaultValue="{}"
        :itemList="[]"
        ></ParkForm>
      </div>
    </el-dialog>
    <el-dialog
      title="修改收付款账单"
      v-if="modifyVisible"
      :before-close="(done) => {
         this.$confirm('表单尚未提交确认关闭？')
          .then(_ => {
            done();
          })
          .catch(_ => {});
        }"
      :visible.sync="modifyVisible"
      width="600px">
      <div>
        <ParkForm
        @onSubmit="fetchModify"
        @onCancel="() => {this.modifyVisible = false}"
        :formList="$formsLabels.paymentForm"
        :options="$store.getters.paymentListOptions"
        :defaultValue="defaultValue"
        :itemList="[]"
        ></ParkForm>
      </div>
    </el-dialog>
    <!--  账单详情-->
    <el-drawer
      title="账单详情"
      custom-class="drawer-r"
      :visible.sync="InfoState"
      size="1186px"
      direction="rtl">
      <HeaderCard :data="financialInfo_header">
        <template #headerCardBtns>
          <div class="btnBox" v-for="(item,i) in financialInfo_header.button" :key="(item,i)" @click="open(item.name)">
            <i class="iconfont" v-html="item.icon"></i>
            <span class="headerCard-btn-name">{{item.name}}</span>
          </div>
        </template>
      </HeaderCard>
      <HeaderInfo type=1 :data="financialInfo_info"></HeaderInfo>
      <div class="drawer-body" style="height: 660px;">
        <BodyCard type=1 :data="financialInfo_body_financial"></BodyCard>
        <BodyCard type=1 :data="financialInfo_body_room"></BodyCard>
        <BodyCard type=2 :data="financialInfo_body_table1"></BodyCard>
        <BodyCard type=2 :data="financialInfo_body_table2"></BodyCard>
        <BodyCard type=2 :data="financialInfo_body_table3"></BodyCard>
        <BodyCard type=2 :data="financialInfo_body_table4"></BodyCard>
        <BodyCard type=2 :data="financialInfo_body_table5"></BodyCard>

      </div>
    </el-drawer>
  </div>
</template>

<script>
import ParkForm from '@/components/ParkForm/index.vue'
import ElCard from 'element-ui/packages/card/src/main'
export default {
  name: 'index',
  components: {
    ElCard,
    ParkForm
  },
  data () {
    return {
      tableData: [],
      activeName: 'first',
      radio: '收款',
      yearList: [
      ],
      infoData: [
      ],
      payment_type: '',
      customer_name: '',
      addVisible: false,
      tamplateFormList: [
        {
          type: 'select',
          label: '模板类型',
          key: 'tamplate',
          placeholder: '请输入',
          rule: [
            { required: true, message: '请选择', trigger: 'change' }
          ],
          options: [
            {
              label: '美食',
              value: 's1'
            }, {
              label: '美食美食',
              value: 's2'
            }
          ]
        }, {
          type: 'input',
          label: '模板名称',
          key: 'i',
          placeholder: '请输入',
          rule: [
            { required: true, message: '请输入模板名称', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ]
        }, {
          type: 'textarea',
          label: '模板描述',
          key: 'i11',
          placeholder: '请输入模板描述',
          rule: [
            { required: true, message: '请输入模板描述', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ]
        }, {
          type: 'upload',
          label: '模板描述',
          key: 'i11',
          placeholder: '请输入模板描述',
          rule: [
            { required: true, message: '请输入模板描述', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ]
        }
      ],
      InfoState: false,
      modifyVisible: false,
      id: '',
      financialInfo_header: {
        title: '收款方：-',
        button: [
          {
            name: '编辑',
            icon: '&#xe62a;'
          },
          {
            name: '删除',
            icon: '&#xe7d1;'
          }
        ]
      },
      financialInfo_info: {
        label: [
          { prop: 'state', label: '账单状态' },
          { prop: 'b', label: '应退金额' },
          { prop: 'c', label: '需退金额' },
          { prop: 'd', label: '应退时间' }
        ],
        tableData: []
      },
      financialInfo_body_financial: {
        title: '账单信息',
        info: []
      },
      financialInfo_body_room: {
        title: '房源信息',
        info: [
          { name: '园区', value: '西港发展中心' },
          { name: '楼宇', value: '协力大厦' },
          { name: '房号', value: '10层302室' }
        ]
      },
      financialInfo_body_table1: {
        title: '收款',
        info: {
          label: [
            { prop: 'a', label: '对方单位名称' },
            { prop: 'b', label: '入账日' },
            { prop: 'c', label: '借贷标' },
            { prop: 'd', label: '发生额' },
            { prop: 'e', label: '匹配金额' },
            { prop: 'f', label: '匹配时间' },
            { prop: 'g', label: '取消匹配时间' },
            { prop: 'h', label: '操作' }
          ],
          tableData: []
        }
      },
      financialInfo_body_table2: {
        title: '付款',
        info: {
          label: [
            { prop: 'a', label: '对方单位名称' },
            { prop: 'b', label: '入账日' },
            { prop: 'c', label: '借贷标' },
            { prop: 'd', label: '发生额' },
            { prop: 'e', label: '匹配金额' },
            { prop: 'f', label: '匹配时间' },
            { prop: 'g', label: '取消匹配时间' },
            { prop: 'h', label: '操作' }
          ],
          tableData: []
        }
      },
      financialInfo_body_table3: {
        title: '结转',
        info: {
          label: [
            { prop: 'a', label: '对方单位' },
            { prop: 'b', label: '转入金额' },
            { prop: 'c', label: '转出金额' },
            { prop: 'd', label: '结转时间' },
            { prop: 'e', label: '作废时间' }
          ],
          tableData: []
        }
      },
      financialInfo_body_table4: {
        title: '开票记录',
        info: {
          label: [
            { prop: 'a', label: '购买方名称' },
            { prop: 'b', label: '发票号码' },
            { prop: 'c', label: '开票金额' },
            { prop: 'd', label: '备注' },
            { prop: 'e', label: '开票时间' },
            { prop: 'f', label: '状态' }
          ],
          tableData: []
        }
      },
      financialInfo_body_table5: {
        title: '调整',
        info: {
          label: [
            { prop: 'a', label: '调整金额' },
            { prop: 'b', label: '调整时间' },
            { prop: 'c', label: '调整类型' },
            { prop: 'd', label: '备注' },
            { prop: 'e', label: '作废调整时间' },
            { prop: 'f', label: '操作' }
          ],
          tableData: []
        }
      },
      defaultValue: {},
      page: {
        page_no: 1,
        total: 0,
        page_size: 5
      }

    }
  },
  methods: {
    handleAddContract () {
      this.addVisible = true
    },
    financialState (data) {
      this.id = data.id
      this.fetchGetInfo(this.id)
      this.InfoState = true
    },
    handleClose () { },
    open (i) {
      if (i === '编辑') {
        this.InfoState = false
        this.fetchGetBack()
        // this.modifyShow = true
        // this.fetchModify(this.id)
      }
      if (i === '删除') {
        this.fetchRemove(this.id)
      }
    },
    fetchAdd (data) { // 添加费用催缴
      let params = {
        ...data
      }
      // params.domain_id = params.domain_id[0]
      this.$https.post(this.$urls.payment.add, params).then((res) => {
        if (res.code === 1000) {
          this.fetchList()
          this.addVisible = false
          this.$message.success('添加成功')
        } else {
          this.$message.error('添加失败')
        }
      })
    },
    fetchRemove (id) { // 删除费用催缴
      this.$confirm('此操作将永久删除该记录, 是否继续?', '提示', {
        distinguishCancelAndClose: true,
        confirmButtonText: '确定',
        cancelButtonText: '取消'
      }).then(() => {
        let params = {
          id: id
        }
        this.$https.post(this.$urls.payment.remove, params).then((res) => {
          if (res.code === 1000) {
            this.fetchList()
            this.InfoState = false
            this.$message.success('删除成功')
          } else {
            this.$message.error('删除失败')
          }
        })
      })
    },
    fetchModify (data) { // 修改费用催缴
      let params = {
        ...data,
        id: this.id
      }
      this.$https.post(this.$urls.payment.modify, params).then((res) => {
        if (res.code === 1000) {
          this.$message.success('修改成功')
          this.defaultValue = {}
          this.fetchList()
          this.modifyVisible = false
        } else {
          this.$message.error('修改失败')
        }
      })
    },
    fetchInfo () { // 获取费用催缴统计信息
      let params = {
        park_id: this.$store.state.form.activePark.domain_id
      }
      this.$https.post(this.$urls.payment.info, params).then((res) => {
        if (res.code === 1000) {
          this.infoData = [
            { name: `应收(${res.need_receive_num})`, value: res.need_receive, chart: res.need_receive_rate, type: 'arrow' },
            { name: '已收', value: res.receive, chart: res.receive_rate, type: 'arrow' },
            { name: `未缴(${res.un_receive_num})`, value: res.un_receive, chart: res.un_receive_rate, type: 'arrow' },
            { name: '租金', value: res.rent, chart: res.rent_rate, type: 'arrow' },
            { name: '物业费', value: res.property_fee, chart: res.property_rate, type: 'arrow' },
            { name: '四表费用', value: res.energy, chart: res.energy_rate, type: 'arrow' },
            { name: '其他', value: res.other, chart: res.other_rate, type: 'arrow' }
          ]
        }
      })
    },
    fetchList () { // 获取费用催缴列表
      let params = {
        ...this.page,
        park_id: this.$store.state.form.activePark.domain_id,
        type: this.payment_type,
        customer_name: this.customer_name
      }
      this.$https.post(this.$urls.payment.get_list, params).then((res) => {
        this.tableData = []
        if (res.code === 1000 && res.list && res.list.length) {
          let list = res.list
          list.forEach(v => {
            v.room = v.building_name + '/' + v.name
          })
          let params = ['type']
          this.$dictionary.tableData(list, params)
          // this.$utils.getRooms(res.list)
          this.page.total = res.total
          this.tableData = res.list
        }
      })
    },
    fetchListSearch () {
      this.page.page_no = 1
      this.fetchList()
    },
    fetchGetBack () {
      let params = {
        id: this.id
      }
      this.$https.post(this.$urls.payment.get_back, params).then(res => {
        if (res.code === 1000) {
          let data = res
          this.defaultValue = data
          this.modifyVisible = true
        } else {
          this.$message.error('获取信息失败')
        }
      })
    },
    fetchGetInfo (id) { // 获取费用催缴信息
      let params = {
        id: id
      }
      this.$https.post(this.$urls.payment.get_info, params).then((res) => {
        this.financialInfo_body_financial.info = []
        if (res.code === 1000) {
          let data = res
          this.financialInfo_body_financial.info = [
            { name: '费用类型', value: data.type },
            { name: '计费周期', value: data.start_ts + '-' + data.end_ts },
            // { name: '账单金额', value:  },
            { name: '创建时间', value: data.create_ts },
            { name: '付款方', value: data.payer },
            // { name: '收款方联系方式', value:  },
            { name: '合同编号', value: data.contract_code },
            // { name: '账单编号', value:  },
            { name: '备注', value: data.memo }
          ]
        }
      })
    },
    handlePageClick (num) { // 点击页码时
      this.page.page_no = num
      this.fetchList()
    }
  },
  created () {
    this.fetchList()
    this.fetchInfo()
  }
}
</script>

<style lang="less" scoped>
  @import '../../assets/style/index.less';
  .el-card{
    margin-bottom: 20px;
  }
</style>
