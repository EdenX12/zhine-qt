<template>
  <div style="margin-top:10px">
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.orderSn" placeholder="请输入订单编号" class="name-input"></el-input>
        <el-input v-model="query.addressName" placeholder="请输入收货人姓名" class="name-input"></el-input>
        <el-input v-model="query.nickName" placeholder="请输入用户昵称" class="name-input"></el-input>
        <el-select v-model="query.orderStatus" placeholder="请选择订单状态" class="type-select">
          <el-option
            v-for="item in orderStatusList"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          ></el-option>
        </el-select>
        <el-select
          v-model="query.shopId"
          placeholder="请选择商户"
          class="type-select"
          :disabled="shopDisabled"
        >
          <el-option
            v-for="item in shopList"
            :key="item.id"
            :label="item.shopName"
            :value="item.id"
          ></el-option>
        </el-select>
        <el-button
          type="primary"
          round
          icon="el-icon-search"
          @click="handleSearch()"
          class="query-button"
        >搜索</el-button>
          <el-button
          type="danger"
          round
          icon="el-icon-refresh"
          @click="resetQuery()"
          class="query-button"
        >重置</el-button>
        <!-- <el-button type="primary" round icon="el-icon-upload2" @click="uploadOrder()">上传发货</el-button> -->
          <!--数据表单-->
      <div class ="filter-container" style="height:50px;">
        <el-row style="top:25%;">
        <el-button
          type="primary"
          icon="el-icon-download"
          @click="downloadOrder()"
          :loading="downloadLoading"
          style="float:left;"
        >待发货订单</el-button>
        <el-upload
          ref="upload"
          action="/admin/excel/uploadOrderShipped"
          :before-upload="beforeUpload"
          multiple
          :on-change="changeHandler"
          :show-file-list="false"
          :http-request="excelUpload"
          style="padding-left:15%;"
        >
          <el-button type="primary" icon="el-icon-upload2">点击上传</el-button>
        </el-upload>
        </el-row>
      </div>
      </div>
      <el-table
        :data="tableData"
        :header-cell-style="tableHeaderColor"
        class="table"
        ref="multipleTable"
        header-cell-class-name="table-header"
        v-loading="tableDataLoading"
        @expand-change="expandSelect"
        :row-key="getRowKeys"
        :expand-row-keys="expands"
      >
        <el-table-column type="expand">
          <template slot-scope="props">
            <el-form label-position="left" class="demo-table-expand">
              <el-form-item label="收货人姓名:">
                <span>{{ props.row.addressName }}</span>
              </el-form-item>
              <el-form-item label="收货人手机号:">
                <span>{{ props.row.addressPhone }}</span>
              </el-form-item>
              <el-form-item label="收货人地址:">
                <span>{{ props.row.addressDetail }}</span>
              </el-form-item>
              <el-form-item label="快递单号:">
                <span>{{ props.row.shippingCode }}</span>
              </el-form-item>
            </el-form>
          </template>
        </el-table-column>
        <el-table-column label="序号" type="index" width="50" align="center"></el-table-column>
        <el-table-column prop="orderSn" label="订单编号" align="center"></el-table-column>
        <el-table-column prop="shopName" label="商户名称" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column prop="nickName" label="用户昵称" align="center"></el-table-column>
        <el-table-column prop="addressName" label="收货人" align="center" width="100px"></el-table-column>
        <el-table-column prop="paymentState" label="付款状态" :formatter="statusFormat" align="center"></el-table-column>
        <el-table-column
          prop="paymentType"
          label="支付方式"
          :formatter="typeFormat"
          align="center"
          width="80px"
        ></el-table-column>
        <el-table-column
          prop="orderStatus"
          label="订单状态"
          :formatter="orderStatusFormat"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="paymentTime"
          label="支付时间"
          :formatter="paymentDateFormat"
          align="center"
        ></el-table-column>
        <el-table-column prop="payAmount" label="实付金额" align="center" width="80px"></el-table-column>
        <el-table-column
          prop="channel"
          label="渠道"
          :formatter="channelFormat"
          align="center"
          width="70px"
        ></el-table-column>
        <el-table-column label="操作" align="center">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-truck"
              circle
              @click="handleMore(scope.$index, scope.row)"
            ></el-button>
            <el-button
              type="primary"
              icon="el-icon-more-outline"
              circle
              @click="handleOrderProduct(scope.$index, scope.row)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <div class="pagination">
        <el-pagination
          background
          layout="total, prev, pager, next"
          :current-page="query.pageNum"
          :page-size="query.pageSize"
          :total="pageTotal"
          @current-change="handlePageChange"
        ></el-pagination>
      </div>
      <!-- 弹出框 -->
      <el-dialog
        :title="dialogMessage"
        :visible.sync="editVisible"
        :center="dialogCentr"
        @close="closeDialog"
      >
        <el-form
          :rules="ruleValidate"
          ref="logisticsForm"
          :model="logisticsForm"
          :inline="true"
          :lable-position="labelPosition"
          label-width="120px"
        >
          <el-form-item label="快递公司" prop="shippingExpressId">
            <el-select v-model="logisticsForm.shippingExpressId" filterable placeholder="请选择快递公司">
              <el-option
                v-for="item in exressList"
                :key="item.id"
                :label="item.ename"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="物流单号" prop="shippingCode">
            <el-input
              v-model="logisticsForm.shippingCode"
              placeholder="请输入物流单号"
              suffix-icon="“xxxx”"
            ></el-input>
          </el-form-item>
          <el-form-item label="发货备注" prop="deliverExplain">
            <el-input
              v-model="logisticsForm.deliverExplain"
              placeholder="请输入发货备注"
              suffix-icon="“xxxx”"
            ></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="updateOrderLogistics()">确定</el-button>
          <el-button @click="editVisible = false">取消</el-button>
        </div>
      </el-dialog>
      <!-- 抽屉 -->
      <el-drawer
        :visible.sync="orderProductDrawer"
        direction="rtl"
        size="70%"
        :destroy-on-close="true"
        :withHeader="false"
        :before-close="handleDrawerClose"
      >
        <div style="margin-left:10px;margin-top:10px;">
          <!--数据-->
          <el-table
            :data="orderProductList"
            :header-cell-style="tableHeaderColor"
            v-loading="innerTableDataLoading"
          >
            <el-table-column label="序号" type="index" width="50" align="center"></el-table-column>
            <el-table-column
              label="商品名称"
              prop="productName"
              align="center"
              :show-overflow-tooltip="true"
            ></el-table-column>
            <el-table-column label="规格名称" prop="productSpecValueName" align="center"></el-table-column>
            <el-table-column label="购买数量" prop="productNumber" align="center"></el-table-column>
          </el-table>
        </div>
      </el-drawer>
      <el-dialog class="express-dialog" title="快递信息" :visible.sync="expressVisible">
        <div>
          <el-steps direction="vertical" :active="0" v-if="expressDataList.length > 0">
            <el-step
              :title="item.ftime"
              :description="item.context"
              icon="el-icon-success"
              v-for="(item,index) in expressDataList"
              :key="index"
            ></el-step>
          </el-steps>
        </div>
      </el-dialog>
    </div>
  </div>
</template>
<script>
export default {
  name: "order",
  data() {
    return {
      query: {
        pageNum: 1,
        pageSize: 10
      },
      tableData: [],
      pageTotal: 0,
      tableDataLoading: true,
      selectOrderId: "",
      labelPosition: "left",
      logisticsForm: {},
      editVisible: false,
      dialogMessage: "订单发货",
      dialogCentr: true,
      ruleValidate: {},
      exressList: [],
      orderStatusList: [
        { id: 0, name: "未付款" },
        { id: 1, name: "已付款待发货" },
        { id: 2, name: "已发货" },
        { id: 3, name: "已确认收货" },
        { id: 4, name: "申请退货退款" },
        { id: 5, name: "已退货退款" },
        { id: 6, name: "超时关闭" },
        { id: -1, name: "已取消" }
      ],
      orderProductDrawer: false,
      orderProductList: [],
      innerTableDataLoading: false,
      expands: [],
      fileExcelList: [],
      expressVisible: false,
      expressDataList: [],
      downloadLoading: false,
      shopList: [],
      shopDisabled: false,
      loginUser: {}
    };
  },
  created() {
    this.handleUserShop();
    this.getTableData();
    this.getShopListData();
  },
  methods: {
    handleUserShop() {
      this.loginUser = JSON.parse(window.localStorage.getItem("user"));
      if (this.loginUser.shopId !== undefined) {
        //处理权限问题
        this.query.shopId = parseInt(this.loginUser.shopId);
        this.shopDisabled = true;
      }
    },
    //获取商户信息
    getShopListData() {
      var url = "/product/listShop";
      var params = { pageNum: 1, pageSize: 10000 };
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.shopList = resp.data.obj.list;
        }
      });
    },
    //上传excel
    excelUpload(params) {
      // eslint-disable-next-line no-console
      console.log(params);
      var url = "/excel/uploadOrderShipped";
      let form = new FormData();
      form.append("file", params.file);
      this.uploadFileRequest(url, form).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.$message.success("处理成功");
          this.tableDataLoading = true;
          this.getTableData();
        }
      });
    },
    beforeUpload(file) {
      const isTextComputer =
        file.type ===
        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet";
      if (!isTextComputer) {
        this.$message.warning("请上传.xlsx格式的excel文件！");
      }
      this.fileExcelList.push(file);
      return isTextComputer;
    },
    changeHandler(file, fileList) {
      //文件状态改变时的钩子，添加文件、上传成功和上传失败时都会被调用
      // eslint-disable-next-line no-console
      console.log(file, fileList);
    },
    //导出待发货订单
    downloadOrder() {
      this.downloadLoading = true;
      var url = "/excel/exportOrderForShipped";
      this.postDownload(url, null).then(resp => {
        if (resp && resp.status == 200) {
          const aLink = document.createElement("a");
          var blob = new Blob([resp.data], {
            type:
              "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
          });
          // //从response的headers中获取filename, 后端response.setHeader("Content-disposition", "attachment; filename=xxxx.docx") 设置的文件名;
          var patt = new RegExp("filename=([^;]+\\.[^\\.;]+);*");
          var contentDisposition = decodeURI(
            resp.headers["content-disposition"]
          );
          var result = patt.exec(contentDisposition);
          var fileName = result[1];
          // eslint-disable-next-line no-useless-escape
          fileName = fileName.replace(/\"/g, "");
          aLink.href = URL.createObjectURL(blob);
          aLink.setAttribute("download", fileName); // 设置下载文件名称
          document.body.appendChild(aLink);
          aLink.click();
          document.body.appendChild(aLink);
        }
        this.downloadLoading = false;
      });
    },
    //关闭弹窗
    closeDialog() {
      this.resetForm("logisticsForm");
      this.logisticsForm = {};
    },
    handleDrawerClose(done) {
      this.orderProductList = [];
      done();
    },
    resetForm(formName) {
      this.$refs[formName].clearValidate();
    },
    //搜索功能
    handleSearch() {
      this.tableDataLoading = true;
      this.getTableData();
    },
    //重置搜索条件
    resetQuery() {
      this.query.nickName = null;
      this.query.orderStatus = null;
      this.query.orderSn = null;
      this.query.addressName = null;
      if (this.loginUser.shopId === undefined) {
        this.query.shopId = null;
      }
      this.tableDataLoading = true;
      this.getTableData();
    },
    //获取数据
    getTableData() {
      var url = "order/listOrderDetail";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.tableData = resp.data.obj.list;
          this.pageTotal = resp.data.obj.total;
          this.tableDataLoading = false;
        }
      });
    },
    //获取快递公司信息
    getExressList() {
      var url = "express/listExpress";
      var params = {};
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.exressList = resp.data.obj;
        }
      });
    },
    updateOrderLogistics() {
      var url = "order/updateOrderLogistics";
      this.postRequest(url, this.logisticsForm).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.$message.success("保存成功");
          this.resetForm("logisticsForm");
          this.getTableData(this.selectProductId);
          this.editVisible = false;
        } else {
          this.$message("保存失败");
        }
      });
    },
    // eslint-disable-next-line no-unused-vars
    handleMore(index, row) {
      if (row.orderStatus == 1) {
        this.getExressList();
        this.editVisible = true;
        this.logisticsForm.id = row.id;
      } else if (row.orderStatus > 1 && row.orderStatus <= 6) {
        var url = "order/queryExpress";
        var params = { shippingCode: row.shippingCode };
        this.postRequest(url, params).then(resp => {
          if (resp && resp.status == 200 && resp.data.status == "200") {
            if (resp.data.obj === null) {
              this.$message({
                showClose: true,
                message: "暂无物流信息",
                type: "warning"
              });
            } else {
              this.expressDataList = resp.data.obj;
              this.expressVisible = true;
            }
          }
        });
      } else {
        this.$message({
          showClose: true,
          message: "订单不是待发货状态",
          type: "warning"
        });
      }
    },
    // eslint-disable-next-line no-unused-vars
    handleOrderProduct(index, row) {
      this.orderProductList = row.orderProductList;
      this.orderProductDrawer = true;
    },
    // eslint-disable-next-line no-unused-vars
    tableHeaderColor(row, rowIndex) {
      return "background-color:#EBEEF5;";
    },
    handlePageChange(val) {
      this.tableDataLoading = true;
      this.$set(this.query, "pageNum", val);
      this.getTableData(this.query);
    },
    // eslint-disable-next-line no-unused-vars
    statusFormat(row, column) {
      if (row.paymentState == 0) {
        return "锁定(支付中)";
      } else if (row.paymentState == 1) {
        return "已支付";
      } else if (row.paymentState == 2) {
        return "待支付";
      } else if (row.paymentState == 3) {
        return "不支付(取消或过期)";
      } else if (row.paymentState == 9) {
        return "已结算到冻结";
      }
    },
    // eslint-disable-next-line no-unused-vars
    orderStatusFormat(row, column) {
      if (row.orderStatus == 0) {
        return "未付款";
      } else if (row.orderStatus == 1) {
        return "已付款待发货";
      } else if (row.orderStatus == 2) {
        return "已发货";
      } else if (row.orderStatus == 3) {
        return "已确认收货";
      } else if (row.orderStatus == 4) {
        return "申请退货退款";
      } else if (row.orderStatus == 5) {
        return "已退货退款";
      } else if (row.orderStatus == 6) {
        return "超时关闭";
      } else if (row.orderStatus == -1) {
        return "已取消";
      }
    },
    // eslint-disable-next-line no-unused-vars
    typeFormat(row, column) {
      if (row.paymentType === null) {
        return null;
      } else if (row.paymentType == 0) {
        return "微信";
      } else if (row.paymentType == 1) {
        return "支付宝";
      }
    },
    // eslint-disable-next-line no-unused-vars
    channelFormat(row, column) {
      if (row.channel === null) {
        return null;
      } else if (row.channel == 0) {
        return "APP";
      } else if (row.channel == 1) {
        return "微信公众号";
      } else if (row.channel == 2) {
        return "小程序";
      }
    },
    // eslint-disable-next-line no-unused-vars
    createDateFormat(row, column) {
      return this.dateFormat(row.createTime);
    },
    // eslint-disable-next-line no-unused-vars
    paymentDateFormat(row, column) {
      return this.dateFormat(row.paymentTime);
    },
    // eslint-disable-next-line no-unused-vars
    dateFormat(date) {
      if (date === null) {
        return null;
      }
      let d = new Date(date); //val 为表格内取到的后台时间
      let month =
        d.getMonth() + 1 < 10 ? "0" + (d.getMonth() + 1) : d.getMonth() + 1;
      let day = d.getDate() < 10 ? "0" + d.getDate() : d.getDate();
      let hours = d.getHours() < 10 ? "0" + d.getHours() : d.getHours();
      let min = d.getMinutes() < 10 ? "0" + d.getMinutes() : d.getMinutes();
      let sec = d.getSeconds() < 10 ? "0" + d.getSeconds() : d.getSeconds();
      let times =
        d.getFullYear() +
        "-" +
        month +
        "-" +
        day +
        " " +
        hours +
        ":" +
        min +
        ":" +
        sec;
      return times;
    },
    getRowKeys(row) {
      return row.id;
    },
    //展开行
    expandSelect(row, expandedRows) {
      var that = this;
      if (expandedRows.length) {
        that.expands = [];
        if (row) {
          that.expands.push(row.id);
        }
      } else {
        that.expands = [];
      }
    }
  }
};
</script>
<style scoped>
.table-product-img {
  display: block;
  margin: auto;
  width: 50px;
  height: 50px;
}
.name-input {
  width: 180px;
  margin-right: 10px;
}
.handle-box {
  margin-bottom: 10px;
  text-align: left;
}
.type-select {
  margin-top: 10px;
  margin-right: 10px;
  width: 180px;
}
.demo-table-expand {
  font-size: 0;
}
.demo-table-expand label {
  width: 90px;
  color: #99a9bf;
}
.demo-table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  width: 50%;
}
.query-button {
  margin-top: 10px;
}
</style>