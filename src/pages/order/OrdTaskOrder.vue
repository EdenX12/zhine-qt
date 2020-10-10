<template>
  <div style="margin-top:10px">
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.orderSn" placeholder="请输入订单编号" class="name-input"></el-input>
        <el-input v-model="query.nickName" placeholder="请输入用户昵称" class="name-input"></el-input>
        <el-input v-model="query.productName" placeholder="请输入商品名称" class="name-input"></el-input>
        <el-select v-model="query.payStatus" placeholder="请选择付款状态" class="type-select">
          <el-option v-for="item in statusList" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>
        <el-button type="primary" round icon="el-icon-search" @click="handleSearch()">搜索</el-button>
        <el-button type="danger" round icon="el-icon-refresh" @click="resetQuery()">重置</el-button>
      </div>
      <!--数据表单-->
      <el-table
        :data="tableData"
        :header-cell-style="tableHeaderColor"
        class="table"
        ref="multipleTable"
        header-cell-class-name="table-header"
        v-loading="tableDataLoading"
      >
        <el-table-column label="序号" type="index" width="50" align="center"></el-table-column>
        <el-table-column prop="orderSn" label="订单编号" align="center" width="200px"></el-table-column>
        <el-table-column prop="nickName" label="用户昵称" align="center"></el-table-column>
        <el-table-column
          prop="productName"
          label="商品名称"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column prop="taskNumber" label="任务份数" align="center" width="80px"></el-table-column>
        <el-table-column prop="payAmount" label="支付金额" align="center" width="80px"></el-table-column>
        <el-table-column prop="payStatus" label="付款状态" :formatter="statusFormat" align="center"></el-table-column>
        <el-table-column
          prop="createTime"
          label="创建时间"
          :formatter="createDateFormat"
          align="center"
        ></el-table-column>
        <el-table-column label="操作" align="center">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-more-outline"
              circle
              @click="handleMore(scope.$index, scope.row)"
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
      editVisible: false,
      statusList: [
        { id: 0, name: "锁定" },
        { id: 1, name: "已支付" },
        { id: 2, name: "待支付" },
        { id: 3, name: "不支付(取消或超时)" }
      ]
    };
  },
  created() {
    this.getTableData();
  },
  methods: {
    //搜索功能
    handleSearch() {
      this.tableDataLoading = true;
      this.getTableData();
    },
    //重置搜索条件
    resetQuery() {
      this.query.nickName = "";
      this.query.orderSn = "";
      this.query.productName = "";
      this.query.payStatus = null;
      this.tableDataLoading = true;
      this.getTableData();
    },
    //获取数据
    getTableData() {
      var url = "order/listTaskOrder";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.tableData = resp.data.obj.list;
          this.pageTotal = resp.data.obj.total;
          this.tableDataLoading = false;
        }
      });
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
      if (row.payStatus == 0) {
        return "锁定";
      } else if (row.payStatus == 1) {
        return "已支付";
      } else if (row.payStatus == 2) {
        return "待支付";
      } else if (row.payStatus == 3) {
        return "不支付(取消或超时)";
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
.upload-demo {
  float: right;
}
</style>