<template>
  <div style="margin-top:10px">
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.realName" placeholder="请输入提现人姓名" class="name-input"></el-input>
        <el-select v-model="query.status" placeholder="请选择提现状态" class="status-select">
          <el-option v-for="item in statusList" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>
        <el-date-picker
          v-model="query.startTime"
          type="datetime"
          placeholder="选择开始时间"
          class="status-select"
        ></el-date-picker>
        <el-date-picker
          v-model="query.endTime"
          type="datetime"
          placeholder="选择结束时间"
          class="status-select"
        ></el-date-picker>
        <el-button type="primary" round icon="el-icon-search" @click="handleSearch()">搜索</el-button>
        <el-button type="danger" round icon="el-icon-refresh" @click="resetQuery()">重置</el-button>
        <el-button
          class="query-button"
          type="primary"
          round
          icon="el-icon-download"
          @click="downloadOrder()"
          :loading="downloadLoading"
        >提现数据</el-button>
        <el-tag type="success" class="el-tag">总计金额：{{sumAmount===null?0:sumAmount}} 元</el-tag>
      </div>
      <!--数据表单-->
      <el-table
        :data="tableData"
        show-summary
        :summary-method="getAmountSummaries"
        :header-cell-style="tableHeaderColor"
        class="table"
        ref="multipleTable"
        header-cell-class-name="table-header"
        v-loading="tableDataLoading"
      >
        <el-table-column label="序号" type="index" width="50" align="center"></el-table-column>
        <el-table-column prop="realName" label="提现人" align="center" width="100px"></el-table-column>
        <el-table-column prop="idCard" label="身份证号" align="center" width="180px"></el-table-column>
        <el-table-column prop="bankName" label="银行名称" align="center" width="150px"></el-table-column>
        <el-table-column prop="cardNum" label="银行卡号" align="center" width="180px"></el-table-column>
        <el-table-column prop="amount" label="提现金额" align="center" width="150px"></el-table-column>
        <el-table-column
          prop="status"
          label="提现状态"
          :formatter="statusFormat"
          align="center"
          width="100px"
        ></el-table-column>
        <el-table-column
          prop="createTime"
          label="创建时间"
          :formatter="createDateFormat"
          align="center"
          width="100px"
        ></el-table-column>
        <el-table-column label="操作" align="center">
          <template slot-scope="scope">
            <div v-if="scope.row.status == 0">
              <el-button round type="success" @click="withdrawPass(scope.$index, scope.row)">审核通过</el-button>
              <el-button round type="danger" @click="withdrawRefuse(scope.$index, scope.row)">审核拒绝</el-button>
            </div>
            <div v-if="scope.row.status == 1">{{statusFormat(scope.row)}}</div>
            <div v-if="scope.row.status == 2">{{statusFormat(scope.row)}}</div>
            <div v-if="scope.row.status == 3">
              <el-button round type="success" @click="withdrawSuccess(scope.$index, scope.row)">打款成功</el-button>
              <el-button round type="danger" @click="withdrawFail(scope.$index, scope.row)">打款失败</el-button>
            </div>
            <div v-if="scope.row.status == 4">{{statusFormat(scope.row)}}</div>
            <div v-if="scope.row.status == 5">{{statusFormat(scope.row)}}</div>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <div class="pagination">
        <el-pagination
          background
          layout="total, sizes, prev, pager, next, jumper"
          :current-page="query.pageNum"
          :page-sizes="[10, 50, 100, 500]"
          :page-size="query.pageSize"
          :total="pageTotal"
          @size-change="handleSizeChange"
          @current-change="handlePageChange"
        ></el-pagination>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: "funcModule",
  data() {
    return {
      dialogCentr: true,
      labelPosition: "left",
      query: {
        pageNum: 1,
        pageSize: 10
      },
      tableData: [],
      pageTotal: 0,
      statusList: [
        { id: 0, name: "待审核" },
        { id: 1, name: "审核通过" },
        { id: 2, name: "审核拒绝" },
        { id: 3, name: "打款中" },
        { id: 4, name: "打款成功" },
        { id: 5, name: "打款失败" }
      ],
      dialogMessage: "",
      saveForm: {},
      addOrUpdate: 0,
      editVisible: false,
      ruleValidate: {},
      tableDataLoading: true,
      sumAmount: "",
      downloadLoading: false
    };
  },
  created() {
    this.getTableData();
  },
  methods: {
    getAmountSummaries(param) {
      const { columns, data } = param;
      const sums = [];
      columns.forEach((column, index) => {
        if (index === 0) {
          sums[index] = "总计";
          return;
        }
        if (index === 1 || index === 2 || index === 3 || index === 4) {
          sums[index] = "N/A";
          return;
        }
        if (index === 5) {
          const values = data.map(item => Number(item[column.property]));
          if (!values.every(value => isNaN(value))) {
            sums[index] = values.reduce((prev, curr) => {
              const value = Number(curr);
              if (!isNaN(value)) {
                return prev + curr;
              } else {
                return prev;
              }
            }, 0);
            sums[index] = sums[index].toFixed(2);
            sums[index] += " 元";
          }
        }
        if (index === 6 || index === 7 || index === 8) {
          sums[index] = "N/A";
          return;
        }
      });
      return sums;
    },
    //导出用户提现列表
    downloadOrder() {
      this.downloadLoading = true;
      var url = "/excel/exportUserWithdraw";
      this.postDownload(url, this.query).then(resp => {
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
    // eslint-disable-next-line no-unused-vars
    withdrawPass(index, row) {
      var status = 1;
      this.updateUserWithdrawStatus(row.id, status);
    },
    // eslint-disable-next-line no-unused-vars
    withdrawRefuse(index, row) {
      var status = 2;
      this.updateUserWithdrawStatus(row.id, status);
    },
    // eslint-disable-next-line no-unused-vars
    withdrawSuccess(index, row) {
      var status = 4;
      this.updateUserWithdrawStatus(row.id, status);
    },
    // eslint-disable-next-line no-unused-vars
    withdrawFail(index, row) {
      var status = 5;
      this.updateUserWithdrawStatus(row.id, status);
    },
    //保存等级
    updateUserWithdrawStatus(id, status) {
      var url = "/user/updateUserWithdrawStatus";
      var params = { id: id, status: status };
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.$message.success("操作成功");
          this.tableDataLoading = true;
          this.getTableData();
        } else {
          this.$message("操作失败");
        }
      });
    },
    getTableData() {
      var url = "/user/listUserWithdraw";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.tableData = resp.data.obj.data.list;
          this.pageTotal = resp.data.obj.data.total;
          this.sumAmount = resp.data.obj.sumAmount;
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
    handleSizeChange(val) {
      this.tableDataLoading = true;
      this.$set(this.query, "pageNum", 1);
      this.$set(this.query, "pageSize", val);
      this.getTableData(this.query);
    },
    //搜索功能
    handleSearch() {
      this.tableDataLoading = true;
      this.getTableData();
    },
    resetQuery() {
      this.query.realName = null;
      this.query.status = null;
      this.query.startTime = null;
      this.query.endTime = null;
      this.query.pageNum = 1;
      this.tableDataLoading = true;
      this.getTableData();
    },
    // eslint-disable-next-line no-unused-vars
    statusFormat(row, column) {
      if (row.status == 0) {
        return "待审核";
      } else if (row.status == 1) {
        return "审核通过";
      } else if (row.status == 2) {
        return "审核拒绝";
      } else if (row.status == 3) {
        return "打款中";
      } else if (row.status == 4) {
        return "打款成功";
      } else if (row.status == 5) {
        return "打款失败";
      }
    },
    // eslint-disable-next-line no-unused-vars
    createDateFormat(row, column) {
      return this.dateFormat(row.createTime);
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
.table-module-img {
  display: block;
  margin: auto;
  width: 50px;
  height: 50px;
}
.handle-box {
  margin-bottom: 10px;
  text-align: left;
}
.name-input {
  width: 200px;
  margin-right: 10px;
}
.status-select {
  margin-right: 10px;
}
.el-tag {
  margin-top: 10px;
  margin-left: 10px;
  font-size: 13px;
}
</style>