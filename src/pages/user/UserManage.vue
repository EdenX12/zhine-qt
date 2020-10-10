<template>
  <div style="margin-top:10px">
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.nickName" placeholder="请输入用户昵称" class="name-input"></el-input>
        <el-select v-model="query.userStatus" placeholder="请选择用户状态" class="status-select">
          <el-option v-for="item in statusList" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>
        <el-select v-model="query.userType" placeholder="请选择用户类型" class="status-select">
          <el-option v-for="item in typeList" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>
        <el-button type="primary" round icon="el-icon-search" @click="handleSearch()">搜索</el-button>
        <el-button type="danger" round icon="el-icon-refresh" @click="resetQuery()">重置</el-button>
        <!-- <el-button type="primary" round icon="el-icon-circle-plus-outline" @click="handleAdd()">添加</el-button> -->
      </div>
      <!--数据表单-->
      <el-table
        :data="tableData"
        :header-cell-style="tableHeaderColor"
        class="table"
        ref="multipleTable"
        header-cell-class-name="table-header"
        v-loading="tableDataLoading"
        @expand-change="userExpandSelect"
        :row-key="getRowKeys"
        :expand-row-keys="expands"
      >
        <el-table-column type="expand">
          <template slot-scope="props">
            <el-form label-position="left" class="demo-table-expand">
              <el-form-item label="订单单数:">
                <span>{{ props.row.orderNum }}</span>
              </el-form-item>
              <el-form-item label="订单金额:">
                <span>{{ props.row.orderAmount }}</span>
              </el-form-item>
              <el-form-item label="用户状态:">
                <span>{{ userStatusFormatRe(props.row.userStatus) }}</span>
              </el-form-item>
              <el-form-item label="创建时间:">
                <span>{{ dateFormat(props.row.createTime) }}</span>
              </el-form-item>
            </el-form>
          </template>
        </el-table-column>
        <el-table-column label="序号" type="index" width="50" align="center"></el-table-column>
        <el-table-column label="头像" width="80px" align="center">
          <template slot-scope="scope">
            <el-image
              class="table-module-img"
              :src="scope.row.userImg"
              :preview-src-list="[scope.row.userImg]"
            ></el-image>
          </template>
        </el-table-column>
        <el-table-column prop="nickName" label="用户昵称" align="center"></el-table-column>
        <el-table-column prop="userPhone" label="手机号码" align="center" width="110px"></el-table-column>
        <el-table-column prop="totalAmount" label="余额" align="center" width="50px"></el-table-column>
        <el-table-column prop="lockAmount" label="冻结金额" align="center" width="80px"></el-table-column>
        <el-table-column prop="rewardBean" label="猎豆余额" align="center" width="100px" sortable></el-table-column>
        <el-table-column prop="taskNum" label="拆单单数" align="center" width="80px"></el-table-column>
        <el-table-column prop="taskNumSum" label="拆单份数" align="center" width="80px"></el-table-column>
        <el-table-column prop="taskAmount" label="拆单金额" align="center" width="80px"></el-table-column>
        <el-table-column
          prop="userType"
          label="用户类型"
          :formatter="userTypeFormat"
          align="center"
          width="80px"
        ></el-table-column>
        <el-table-column prop="userLevelTypeName" label="等级" align="center" width="100px"></el-table-column>
        <el-table-column label="操作" align="center">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-more-outline"
              circle
              @click="handleMore(scope.$index, scope.row)"
            ></el-button>
            <el-button
              type="primary"
              icon="el-icon-edit"
              circle
              @click="handleEdit(scope.$index, scope.row)"
            ></el-button>
            <!-- <el-button
              type="danger"
              icon="el-icon-delete"
              circle
              @click="handleDelete(scope.$index, scope.row)"
            ></el-button>-->
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
      <!-- 弹出框 -->
      <el-dialog
        :title="dialogMessage"
        :visible.sync="editVisible"
        :center="dialogCentr"
        @close="closeDialog"
      >
        <el-form
          :rules="ruleValidate"
          ref="saveForm"
          :model="saveForm"
          :inline="true"
          :lable-position="labelPosition"
          label-width="120px"
        >
          <el-form-item label="账户状态" prop="moduleStatus">
            <el-select v-model="saveForm.userStatus" placeholder="请选择状态" suffix-icon="“xxxx”">
              <el-option
                v-for="item in statusList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="用户等级" prop="moduleStatus">
            <el-select v-model="saveForm.userLevelType" placeholder="请选择状态" suffix-icon="“xxxx”">
              <el-option
                v-for="item in userLevelList"
                :key="item.id"
                :label="item.levelName"
                :value="item.levelType"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="saveUser()">确定</el-button>
          <el-button @click="editVisible = false">取消</el-button>
        </div>
      </el-dialog>
      <!-- 抽屉列表 -->
      <el-drawer
        :visible.sync="couponDrawer"
        direction="rtl"
        size="60%"
        :destroy-on-close="true"
        :withHeader="false"
        :before-close="handleDrawerClose"
      >
        <div style="margin-left:10px;margin-top:10px;">
          <!--搜索-->
          <!-- <div class="handle-box">
            <el-button
              type="primary"
              round
              icon="el-icon-circle-plus-outline"
              @click="handleCouponAdd()"
            >添加优惠券</el-button>
          </div>-->
          <!--数据-->
          <el-table
            :data="couponList"
            :header-cell-style="tableHeaderColor"
            v-loading="innerTableDataLoading"
            @expand-change="expandSelect"
            :row-key="getRowKeys"
            :expand-row-keys="expands"
          >
            <el-table-column type="expand">
              <template slot-scope="props">
                <el-form
                  ref="detailDataForm"
                  :model="detailDataForm"
                  label-position="right"
                  inline
                  class="demo-table-expand"
                >
                  <el-form-item label="商品名称">
                    <span>{{ props.row.productName }}</span>
                  </el-form-item>
                  <el-form-item label="优惠券名称">
                    <span>{{ detailDataForm.couponName }}</span>
                  </el-form-item>
                  <el-form-item label="优惠券金额">
                    <span>￥{{ detailDataForm.couponAmount }}</span>
                  </el-form-item>
                  <el-form-item label="开始时间">
                    <span>{{ detailDataForm.startDateFormat }}</span>
                  </el-form-item>
                  <el-form-item label="结束时间">
                    <span>{{ detailDataForm.endDateFormat }}</span>
                  </el-form-item>
                  <el-form-item label="券总数量">
                    <span>{{ detailDataForm.couponQuantity}}</span>
                  </el-form-item>
                  <el-form-item label="领取限制">
                    <span>{{ detailDataForm.perLimit}}</span>
                  </el-form-item>
                </el-form>
              </template>
            </el-table-column>
            <el-table-column label="序号" type="index" width="50" align="center"></el-table-column>
            <el-table-column
              prop="productName"
              label="商品名称"
              align="center"
              :show-overflow-tooltip="true"
            ></el-table-column>
            <el-table-column prop="couponType" label="类型" :formatter="typeFormat" align="center"></el-table-column>
            <el-table-column
              prop="couponStatus"
              label="状态"
              :formatter="statusFormat"
              align="center"
            ></el-table-column>
            <el-table-column label="操作" align="center">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  circle
                  @click="handleCouponEdit(scope.$index, scope.row)"
                ></el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  circle
                  @click="handleCouponDelete(scope.$index, scope.row)"
                ></el-button>
              </template>
            </el-table-column>
          </el-table>
        </div>
        <!-- 嵌套抽屉 -->
        <el-drawer
          size="50%"
          :append-to-body="true"
          :before-close="handleInnerDrawerClose"
          :visible.sync="innerDrawer"
          :show-close="false"
          :withHeader="false"
        >
          <div style="text-align: center;">
            <span class="inner-drawer-tittle">{{innerDrawerTittle}}</span>
          </div>
          <el-form
            :rules="ruleValidate"
            ref="saveUserCouponForm"
            :model="saveUserCouponForm"
            :inline="true"
            :lable-position="labelPosition"
            label-width="120px"
          ></el-form>
          <div class="demo-drawer__footer" style="text-align:center;">
            <el-button type="primary" @click="saveShopCoupon()">确 定</el-button>
            <el-button @click="innerDrawerClose()">取 消</el-button>
          </div>
        </el-drawer>
      </el-drawer>
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
        { id: 0, name: "可用" },
        { id: 1, name: "不可用" }
      ],
      typeList: [
        { id: 1, name: "普通客户(拆家)" },
        { id: 2, name: "商家" }
      ],
      dialogImageUrl: "",
      imgDialogVisible: false,
      fileLists: [],
      dialogMessage: "",
      saveForm: {},
      addOrUpdate: 0,
      editVisible: false,
      ruleValidate: {},
      tableDataLoading: true,
      userLevelList: [],
      selectUserId: "",
      couponDrawer: false,
      couponList: [],
      innerTableDataLoading: true,
      saveUserCouponForm: {},
      innerDrawer: false,
      innerDrawerTittle: "",
      userCouponAddOrUpdate: 0,
      detailDataForm: {},
      expands: []
    };
  },
  created() {
    this.getTableData();
    this.getUserLevelList();
  },
  methods: {
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
          this.getCouponDetailData(row.couponId, row.couponType);
        }
      } else {
        that.expands = [];
        this.detailDataForm = {};
      }
    },
    //展开行
    userExpandSelect(row, expandedRows) {
      var that = this;
      if (expandedRows.length) {
        that.expands = [];
        if (row) {
          that.expands.push(row.id);
        }
      } else {
        that.expands = [];
      }
    },
    //用户优惠券详情
    getCouponDetailData(id, type) {
      var url = "/user/getUserCouponDetail";
      var params = { couponId: id, couponType: type };
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.detailDataForm = resp.data.obj;
        } else {
          this.$message(resp.data.msg);
        }
      });
    },
    //关闭抽屉
    handleDrawerClose(done) {
      this.couponList = [];
      this.innerTableDataLoading = true;
      done();
    },
    /* eslint-disable no-unused-vars */
    handleInnerDrawerClose(done) {
      this.$confirm("确定关闭吗？")
        .then(_ => {
          this.resetForm("saveUserCouponForm");
          this.saveUserCouponForm = {};
          done();
        })
        .catch(_ => {});
    },
    //保存角色
    saveUser() {
      this.$refs.saveForm.validate(valid => {
        if (valid) {
          var url = "";
          if (this.addOrUpdate == 0) {
            url = "/user/insertUser";
          } else if (this.addOrUpdate == 1) {
            url = "/user/updateUser";
          }
          this.saveForm.userImg = this.fileLists[0].url;
          var params = this.saveForm;
          this.postRequest(url, params).then(resp => {
            if (resp && resp.status == 200 && resp.data.status == "200") {
              this.$message.success("保存成功");
              this.editVisible = false;
              this.getTableData();
            } else {
              this.$message("保存失败");
            }
          });
        }
      });
    },
    //添加弹出
    handleAdd() {
      this.dialogMessage = "添加用户";
      this.addOrUpdate = 0;
      this.editVisible = true;
    },
    //关闭弹窗
    closeDialog() {
      this.resetForm("saveForm");
      this.saveForm = {};
      this.fileLists = [];
    },
    resetForm(formName) {
      this.$refs[formName].clearValidate();
    },
    getTableData() {
      var url = "/user/listUser";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.tableData = resp.data.obj.list;
          this.pageTotal = resp.data.obj.total;
          this.tableDataLoading = false;
        }
      });
    },
    getUserLevelList() {
      var url = "/user/listUserLevel";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.userLevelList = resp.data.obj;
        }
      });
    },
    //商户优惠券列表数据
    getCouponTableData(id) {
      var url = "/user/listUserCoupon";
      var params = { userId: id };
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.couponList = resp.data.obj;
          this.innerTableDataLoading = false;
        }
      });
    },
    //查看用户优惠券列表
    // eslint-disable-next-line no-unused-vars
    handleMore(index, row) {
      this.selectUserId = row.id;
      this.couponDrawer = true;
      this.innerTableDataLoading = true;
      this.getCouponTableData(row.id);
    },
    //编辑弹出
    handleEdit(index, row) {
      this.dialogMessage = "编辑用户";
      this.addOrUpdate = 1;
      this.idx = index;
      this.saveForm = JSON.parse(JSON.stringify(row));
      this.fileLists = [
        {
          name: "",
          url: row.userImg
        }
      ];
      this.editVisible = true;
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
      this.query.nickName = null;
      this.query.userStatus = null;
      this.query.userType = null;
      this.tableDataLoading = true;
      this.getTableData();
    },
    // eslint-disable-next-line no-unused-vars
    statusFormat(row, column) {
      if (row.couponStatus == 0) {
        return "未使用";
      } else if (row.couponStatus == 1) {
        return "已使用";
      } else if (row.couponStatus == 2) {
        return "过期";
      }
    },
    userStatusFormat(row, column) {
      if (row.userStatus == 0) {
        return "可用";
      } else if (row.userStatus == 1) {
        return "不可用";
      }
    },
    userStatusFormatRe(userStatus) {
      if (userStatus == 0) {
        return "可用";
      } else if (userStatus == 1) {
        return "不可用";
      }
    },
    // eslint-disable-next-line no-unused-vars
    typeFormat(row, column) {
      if (row.couponType == 0) {
        return "任务金";
      } else if (row.couponType == 1) {
        return "商铺券";
      }
    },
    userTypeFormat(row, column) {
      if (row.userType == 1) {
        return "普通客户(拆家)";
      } else if (row.userType == 2) {
        return "商家";
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
    },
    //删除图片
    // eslint-disable-next-line no-unused-vars
    handleRemove(file, fileList) {
      this.fileLists = [];
    },
    //查看大图
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.imgDialogVisible = true;
    },
    //图片上传成功
    // eslint-disable-next-line no-unused-vars
    success(response, file, fileList) {
      this.fileLists = [
        {
          name: file.name,
          url: response.url
        }
      ];
    },
    // eslint-disable-next-line no-unused-vars
    handleExceed(files, fileList) {
      this.$message.warning(`请最多上传一个图片。`);
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
.demo-table-expand label {
  width: 90px;
  color: #99a9bf;
}
.demo-table-expand .el-form-item {
  margin-right: 0;
  margin-bottom: 0;
  width: 70%;
}
</style>