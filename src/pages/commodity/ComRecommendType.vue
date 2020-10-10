<template>
  <div style="margin-top:10px;">
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.recommendTitle" placeholder="请输入推荐分类名称" class="name-input"></el-input>
        <el-select v-model="query.pageStyle" placeholder="请选择排版风格" class="status-select">
          <el-option label="推荐" value="0"></el-option>
          <el-option label="常规" value="1"></el-option>
        </el-select>
        <el-select v-model="query.recommendStatus" placeholder="请选择推荐分类状态" class="status-select">
          <el-option
            v-for="item in recommendStatusList"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          ></el-option>
        </el-select>
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
        <el-table-column
          prop="recommendTitle"
          label="推荐分类名称"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column
          prop="pageStyle"
          label="排版风格"
          :formatter="styleFormat"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column
          prop="recommendStatus"
          label="状态"
          :formatter="statusFormat"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column prop="sOrder" label="排序" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column prop="createTime" label="创建时间" :formatter="dateFormat" align="center"></el-table-column>
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
          <el-form-item label="推荐分类名称" prop="recommendTitle">
            <el-input
              v-model="saveForm.recommendTitle"
              placeholder="请输入推荐分类名称"
              suffix-icon="“xxxx”"
            ></el-input>
          </el-form-item>
          <el-form-item label="排版风格" prop="pageStyle">
            <el-select v-model="saveForm.pageStyle" placeholder="请选择排版风格" class="status-select">
              <el-option label="推荐" value="0"></el-option>
              <el-option label="常规" value="1"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="推荐分类排序" prop="sOrder">
            <el-input v-model="saveForm.sOrder" placeholder="请输入排序" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="推荐分类状态" prop="sorder">
            <el-select v-model="saveForm.recommendStatus" placeholder="请选择推荐分类状态">
              <el-option
                v-for="item in recommendStatusList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="saveRecommendType()">确定</el-button>
          <el-button @click="editVisible = false">取消</el-button>
        </div>
      </el-dialog>
      <!-- 抽屉列表 -->
      <el-drawer
        :visible.sync="recommendProductDrawer"
        direction="rtl"
        size="70%"
        :destroy-on-close="true"
        :withHeader="false"
        :before-close="handleDrawerClose"
      >
        <div style="margin-left:10px;margin-top:10px;">
          <!--搜索-->
          <div class="handle-box">
            <el-input
              v-model="recommendProductQuery.productName"
              placeholder="请输入商品名称"
              class="name-input"
            ></el-input>
            <el-button type="primary" round icon="el-icon-search" @click="handleProductSearch()">搜索</el-button>
            <el-button type="danger" round icon="el-icon-refresh" @click="resetProductQuery()">重置</el-button>
            <el-button
              type="primary"
              round
              icon="el-icon-circle-plus-outline"
              @click="handleRecommendProductAdd()"
            >添加</el-button>
          </div>
          <!--数据-->
          <el-table
            :data="recommendProductList"
            :header-cell-style="tableHeaderColor"
            v-loading="innerTableDataLoading"
          >
            <el-table-column label="序号" type="index" width="50" align="center"></el-table-column>
            <el-table-column label="图片" align="center" width="80px">
              <template slot-scope="scope">
                <el-image
                  class="table-module-img"
                  :src="scope.row.productImg"
                  :preview-src-list="[scope.row.productImg]"
                ></el-image>
              </template>
            </el-table-column>
            <el-table-column
              prop="productName"
              label="商品名称"
              align="center"
              :show-overflow-tooltip="true"
            ></el-table-column>
            <el-table-column
              prop="isOnFace"
              label="是否在首页显示"
              align="center"
              :formatter="inOnFaceFormatter"
            ></el-table-column>
            <el-table-column prop="sOrder" label="排序" align="center"></el-table-column>
            <el-table-column prop="createTime" label="创建时间" :formatter="dateFormat" align="center"></el-table-column>
            <el-table-column label="操作" align="center">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  circle
                  @click="handleRecommendProductEdit(scope.$index, scope.row)"
                ></el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  circle
                  @click="handleRecommendProductDelete(scope.$index, scope.row)"
                ></el-button>
              </template>
            </el-table-column>
          </el-table>
          <div class="pagination">
            <el-pagination
              background
              layout="total, prev, pager, next"
              :current-page="recommendProductQuery.pageNum"
              :page-size="recommendProductQuery.pageSize"
              :total="recommendPageTotal"
              @current-change="handlePageChangeRecommemd"
            ></el-pagination>
          </div>
        </div>
        <!--嵌套抽屉-->
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
            ref="saveRecommendProductForm"
            :model="saveRecommendProductForm"
            :inline="true"
            :lable-position="labelPosition"
            label-width="120px"
          >
            <el-form-item label="商品选择" prop="productId">
              <el-select
                v-model="saveRecommendProductForm.productId"
                :disabled="productDisabled"
                filterable
                placeholder="请选择商品"
              >
                <el-option
                  v-for="item in productList"
                  :key="item.id"
                  :label="item.productName"
                  :value="item.id"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="排序" prop="sOrder">
              <el-input
                v-model="saveRecommendProductForm.sOrder"
                placeholder="请输入排序"
                suffix-icon="“xxxx”"
              ></el-input>
            </el-form-item>
            <el-form-item label="首页展示" prop="isOnFace">
              <el-select
                v-model="saveRecommendProductForm.isOnFace"
                placeholder="请选择推荐分类"
                class="status-select"
              >
                <el-option label="不显示" value="0"></el-option>
                <el-option label="显示" value="1"></el-option>
              </el-select>
            </el-form-item>
          </el-form>
          <div class="demo-drawer__footer" style="text-align:center;">
            <el-button type="primary" @click="saveRecommendProduct()">确 定</el-button>
            <el-button @click="innerDrawerClose()">取 消</el-button>
          </div>
        </el-drawer>
      </el-drawer>
    </div>
  </div>
</template>
<script>
export default {
  name: "banner",
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
      recommendPageTotal: 0,
      pageStyleList: [
        { id: 0, typeName: "推荐" },
        { id: 1, typeName: "常规" }
      ],
      recommendStatusList: [
        { id: 0, name: "创建" },
        { id: 1, name: "发布" },
        { id: 2, name: "下架" }
      ],
      dialogImageUrl: "",
      imgDialogVisible: false,
      fileLists: [],
      dialogMessage: "",
      saveForm: {},
      addOrUpdate: 0,
      editVisible: false,
      ruleValidate: {
        recommendTitle: [
          { required: true, message: "请输入推荐分类名称", trigger: "blur" }
        ],
        recommendStatus: [
          { required: true, message: "请选择推荐分类类型", trigger: "change" }
        ],
        sOrder: [
          { required: true, message: "请输入推荐分类排序", trigger: "blur" }
        ]
      },
      tableDataLoading: true,
      recommendProductDrawer: false,
      innerTableDataLoading: true,
      selectRecommendTypeId: "",
      recommendProductQuery: { pageNum: 1, pageSize: 5 },
      recommendProductList: [],
      innerDrawerTittle: "",
      saveRecommendProductForm: {},
      recommendProductAddOrUpdate: 0,
      innerDrawer: false,
      productList: [],
      productDisabled: false
    };
  },
  created() {
    this.getTableData();
    this.getProductList();
  },
  methods: {
    //删除
    handleDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/product/deleteRecommendType";
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
    //保存推荐分类
    saveRecommendType() {
      this.$refs.saveForm.validate(valid => {
        if (valid) {
          var url = "";
          if (this.addOrUpdate == 0) {
            url = "/product/insertRecommendType";
          } else if (this.addOrUpdate == 1) {
            url = "/product/updateRecommendType";
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
    //保存推荐商品
    saveRecommendProduct() {
      var url = "";
      if (this.recommendProductAddOrUpdate == 0) {
        url = "/product/insertRecommendProduct";
      } else if (this.recommendProductAddOrUpdate == 1) {
        url = "/product/updateRecommendProduct";
      }
      this.saveRecommendProductForm.sOrder = parseInt(
        this.saveRecommendProductForm.sOrder
      );
      this.saveRecommendProductForm.recommendTypeId = this.selectRecommendTypeId;
      var params = this.saveRecommendProductForm;
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.$message.success("保存成功");
          this.resetForm("saveRecommendProductForm");
          this.saveRecommendProductForm = {};
          this.innerDrawer = false;
          this.innerTableDataLoading = true;
          this.getRecommendProductTableData(this.selectRecommendTypeId);
        } else {
          this.$message("保存失败");
        }
      });
    },
    //添加弹出
    handleAdd() {
      this.dialogMessage = "添加推荐分类";
      this.addOrUpdate = 0;
      this.editVisible = true;
    },
    //添加弹出
    handleRecommendProductAdd() {
      this.innerDrawerTittle = "添加推荐商品";
      this.recommendProductAddOrUpdate = 0;
      this.productDisabled = false;
      this.innerDrawer = true;
    },
    //查看推荐分类下商品列表
    // eslint-disable-next-line no-unused-vars
    handleMore(index, row) {
      this.selectRecommendTypeId = row.id;
      this.recommendProductDrawer = true;
      this.innerTableDataLoading = true;
      this.getRecommendProductTableData(row.id);
    },
    handleRecommendProductEdit(index, row) {
      this.innerDrawerTittle = "编辑推荐商品";
      this.recommendProductAddOrUpdate = 1;
      this.saveRecommendProductForm = JSON.parse(JSON.stringify(row));
      this.productDisabled = true;
      this.innerDrawer = true;
    },
    //关闭弹窗
    closeDialog() {
      this.resetForm("saveForm");
      this.saveForm = {};
    },
    handleDrawerClose(done) {
      this.recommendProductList = [];
      this.recommendProductQuery.pageNum = 1;
      done();
    },
    /* eslint-disable no-unused-vars */
    handleInnerDrawerClose(done) {
      this.$confirm("确定关闭吗？")
        .then(_ => {
          this.resetForm("saveRecommendProductForm");
          this.saveRecommendProductForm = {};
          done();
        })
        .catch(_ => {});
    },
    innerDrawerClose() {
      this.innerDrawer = false;
      this.resetForm("saveRecommendProductForm");
      this.saveRecommendProductForm = {};
    },
    resetForm(formName) {
      this.$refs[formName].clearValidate();
    },
    getTableData() {
      var url = "/product/listRecommendTypePage";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.tableData = resp.data.obj.list;
          this.pageTotal = resp.data.obj.total;
          this.tableDataLoading = false;
        }
      });
    },
    //推荐分类下商品列表
    getRecommendProductTableData(id) {
      var url = "/product/listRecommendProduct";
      this.recommendProductQuery.recommendTypeId = id;
      this.postRequest(url, this.recommendProductQuery).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.recommendProductList = resp.data.obj.list;
          this.recommendPageTotal = resp.data.obj.total;
          this.innerTableDataLoading = false;
        }
      });
    },
    //商品列表
    getProductList() {
      var url = "product/listUsableProduct";
      var param = {};
      this.postRequest(url, param).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.productList = resp.data.obj;
        }
      });
    },
    //编辑弹出
    handleEdit(index, row) {
      this.dialogMessage = "编辑推荐分类";
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
    handlePageChangeRecommemd(val) {
      this.innerTableDataLoading = true;
      this.$set(this.recommendProductQuery, "pageNum", val);
      this.getRecommendProductTableData(this.selectRecommendTypeId);
    },
    //搜索功能
    handleSearch() {
      this.tableDataLoading = true;
      this.query.pageNum = 1;
      this.getTableData();
    },
    handleProductSearch() {
      this.innerTableDataLoading = true;
      this.recommendProductQuery.pageNum = 1;
      this.getRecommendProductTableData(this.selectRecommendTypeId);
    },
    resetQuery() {
      this.query.recommendTitle = null;
      this.query.pageStyle = null;
      this.query.recommendStatus = null;
      this.getTableData();
    },
    resetProductQuery() {
      this.recommendProductQuery.productName = null;
    },
    // eslint-disable-next-line no-unused-vars
    styleFormat(row, column) {
      if (row.pageStyle == 0) {
        return "推荐";
      } else if (row.pageStyle == 1) {
        return "常规";
      }
    },
    // eslint-disable-next-line no-unused-vars
    statusFormat(row, column) {
      if (row.recommendStatus == 0) {
        return "创建";
      } else if (row.recommendStatus == 1) {
        return "发布";
      } else if (row.recommendStatus == 2) {
        return "下架";
      }
    },
    // eslint-disable-next-line no-unused-vars
    inOnFaceFormatter(row, column) {
      if (row.isOnFace == 0) {
        return "不显示";
      } else if (row.isOnFace == 1) {
        return "显示";
      }
    },
    // eslint-disable-next-line no-unused-vars
    dateFormat(row, column) {
      if (row.createTime === null) {
        return null;
      }
      let d = new Date(row.createTime); //val 为表格内取到的后台时间
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
    // eslint-disable-next-line no-unused-vars
    handleExceed(files, fileList) {
      this.$message.warning(`请最多上传一个图片。`);
    }
  }
};
</script>
<style scoped>
.table-banner-img {
  display: block;
  margin: auto;
  width: 150px;
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
.inner-drawer-tittle {
  font-size: 18px;
  color: #303133;
  line-height: 50px;
}
</style>