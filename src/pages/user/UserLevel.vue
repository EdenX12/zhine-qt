<template>
  <div style="margin-top:10px">
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.levelName" placeholder="请输入等级名称" class="name-input"></el-input>
        <el-button type="primary" round icon="el-icon-search" @click="handleSearch()">搜索</el-button>
        <el-button type="danger" round icon="el-icon-refresh" @click="resetQuery()">重置</el-button>
        <el-button type="primary" round icon="el-icon-circle-plus-outline" @click="handleAdd()">添加</el-button>
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
        <el-table-column prop="levelType" label="等级" align="center"></el-table-column>
        <el-table-column prop="levelName" label="等级名称" align="center"></el-table-column>
        <el-table-column prop="minNumber" label="达标拆豆" align="center"></el-table-column>
        <el-table-column prop="maxNumber" label="最大拆豆" align="center"></el-table-column>
        <el-table-column prop="buyNumber" label="最大购买任务线数" align="center"></el-table-column>
        <el-table-column prop="productNumber" label="最多并行商品数量" align="center"></el-table-column>
        <el-table-column label="操作" align="center">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              circle
              @click="handleEdit(scope.$index, scope.row)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              circle
              @click="handleDelete(scope.$index, scope.row)"
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
            label-width="130px"
          >
            <el-form-item label="等级" prop="levelType">
              <el-input v-model="saveForm.levelType" placeholder="请输入等级"></el-input>
            </el-form-item>
            <el-form-item label="等级名称" prop="levelName">
              <el-input v-model="saveForm.levelName" placeholder="请输入等级名称"></el-input>
            </el-form-item>
            <el-form-item label="达标单数" prop="minNumber">
              <el-input v-model="saveForm.minNumber" placeholder="请输入达标单数"></el-input>
            </el-form-item>
            <el-form-item label="最大单数" prop="maxNumber">
              <el-input v-model="saveForm.maxNumber" placeholder="请输入最大单数"></el-input>
            </el-form-item>
            <el-form-item label="最大购买任务线数" prop="buyNumber">
              <el-input v-model="saveForm.buyNumber" placeholder="请输入最大购买任务线数"></el-input>
            </el-form-item>
            <el-form-item label="最多并行商品数量" prop="productNumber">
              <el-input v-model="saveForm.productNumber" placeholder="请输入最多并行商品数量"></el-input>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button type="primary" @click="saveUserLevel()">确定</el-button>
            <el-button @click="editVisible = false">取消</el-button>
          </div>
        </el-dialog>
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
        { id: 0, name: "可用" },
        { id: 1, name: "不可用" }
      ],
      typeList: [
        { id: 1, name: "普通客户" },
        { id: 2, name: "商家" },
        { id: 3, name: "拆家" },
        { id: 4, name: "拆家和商家" }
      ],
      dialogMessage: "",
      saveForm: {},
      addOrUpdate: 0,
      editVisible: false,
      ruleValidate: {},
      tableDataLoading: true
    };
  },
  created() {
    this.getTableData();
  },
  methods: {
    //保存等级
    saveUserLevel() {
      this.$refs.saveForm.validate(valid => {
        if (valid) {
          var url = "";
          if (this.addOrUpdate == 0) {
            url = "/user/insertUserLevel";
          } else if (this.addOrUpdate == 1) {
            url = "/user/updateUserLevel";
          }
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
    handleDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/user/deleteUserLevel";
          var params = { id: row.id };
          this.postRequest(url, params).then(resp => {
            if (resp && resp.status == 200 && resp.data.status == "200") {
              this.$message.success("删除成功");
              this.getTableData();
            } else {
              this.$message("删除失败");
            }
          });
        })
        .catch(() => {});
    },
    //添加弹出
    handleAdd() {
      this.dialogMessage = "添加等级";
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
      var url = "/user/listUserLevelPage";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.tableData = resp.data.obj.list;
          this.pageTotal = resp.data.obj.total;
          this.tableDataLoading = false;
        }
      });
    },
    //编辑弹出
    handleEdit(index, row) {
      this.dialogMessage = "编辑等级";
      this.addOrUpdate = 1;
      this.idx = index;
      this.saveForm = JSON.parse(JSON.stringify(row));
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
    //搜索功能
    handleSearch() {
      this.getTableData();
    },
    resetQuery() {
      this.query.levelName = null;
      this.getTableData();
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
</style>