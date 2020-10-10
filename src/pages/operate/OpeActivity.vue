<template>
  <div style="margin-top:10px">
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.actName" placeholder="请输入活动名称" class="name-input"></el-input>
        <el-select v-model="query.actStatus" placeholder="请选择活动状态" class="status-select">
          <el-option
            v-for="item in actStatusList"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          ></el-option>
        </el-select>
        <el-select v-model="query.flag" placeholder="请选择倒计时标记" class="status-select">
          <el-option v-for="item in flagList" :key="item.id" :label="item.name" :value="item.id"></el-option>
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
        <el-table-column label="图片" width="80px" align="center">
          <template slot-scope="scope">
            <el-image
              class="table-module-img"
              :src="scope.row.pictureUrl"
              :preview-src-list="[scope.row.pictureUrl]"
            ></el-image>
          </template>
        </el-table-column>
        <el-table-column prop="actName" label="活动名称" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column prop="actType" label="活动类型" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column
          prop="actStatus"
          label="状态"
          :formatter="statusFormat"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column prop="sOrder" label="排序" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column
          prop="flag"
          label="是否需要倒计时"
          :formatter="flagFormat"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
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
          <el-form-item label="活动名称" prop="actName">
            <el-input v-model="saveForm.actName" placeholder="请输入活动名称" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="活动类型" prop="actType">
            <el-input v-model="saveForm.actType" placeholder="请输入活动类型" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="活动排序" prop="sOrder">
            <el-input v-model="saveForm.sOrder" placeholder="请输入排序" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="活动状态" prop="actStatus">
            <el-select v-model="saveForm.actStatus" placeholder="请选择状态">
              <el-option
                v-for="item in actStatusList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="活动倒计时标记" prop="flag">
            <el-select v-model="saveForm.flag" placeholder="请选择状态">
              <el-option
                v-for="item in flagList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="图片上传" prop="pictureUrl" style="width:100%;">
            <el-upload
              name="file"
              :limit="1"
              action="/admin/upload/file"
              list-type="picture-card"
              :file-list="fileLists"
              :on-preview="handlePictureCardPreview"
              :on-success="success"
              :on-remove="handleRemove"
              :on-exceed="handleExceed"
            >
              <i class="el-icon-plus"></i>
            </el-upload>
            <el-dialog :visible.sync="imgDialogVisible" size="full" :modal="false" title="查看大图片">
              <img width="60%" :src="dialogImageUrl" alt />
            </el-dialog>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="saveActivity()">确定</el-button>
          <el-button @click="editVisible = false">取消</el-button>
        </div>
      </el-dialog>
      <!-- 抽屉列表 -->
      <el-drawer
        :visible.sync="activityProductDrawer"
        direction="rtl"
        size="80%"
        :destroy-on-close="true"
        :withHeader="false"
        :before-close="handleDrawerClose"
      >
        <div style="margin-left:10px;margin-top:10px;">
          <!--搜索-->
          <div class="handle-box">
            <el-input v-model="productQuery.productName" placeholder="请输入商品名称" class="name-input"></el-input>
            <el-button type="primary" round icon="el-icon-search" @click="handleProductSearch()">搜索</el-button>
            <el-button type="danger" round icon="el-icon-refresh" @click="resetProductQuery()">重置</el-button>
            <el-button
              type="primary"
              round
              icon="el-icon-circle-plus-outline"
              @click="handleProductAdd()"
            >添加</el-button>
          </div>
          <!--数据-->
          <el-table
            :data="activityProductList"
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
              width="110px"
            ></el-table-column>
            <el-table-column label="任务金促销价" align="center" width="110px">
              <template slot-scope="scope">￥{{scope.row.price}}</template>
            </el-table-column>
            <el-table-column label="商品划线价格" align="center" width="110px">
              <template slot-scope="scope">￥{{scope.row.scribingPrice}}</template>
            </el-table-column>
            <el-table-column prop="sorder" label="排序" align="center" width="50px"></el-table-column>
            <el-table-column
              prop="startDate"
              label="开始时间"
              :formatter="startDateFormat"
              align="center"
            ></el-table-column>
            <el-table-column prop="endDate" label="结束时间" :formatter="endDateFormat" align="center"></el-table-column>
            <el-table-column prop="createTime" label="创建时间" :formatter="dateFormat" align="center"></el-table-column>
            <el-table-column label="操作" align="center">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  circle
                  @click="handleProductEdit(scope.$index, scope.row)"
                ></el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  circle
                  @click="handleProductDelete(scope.$index, scope.row)"
                ></el-button>
              </template>
            </el-table-column>
          </el-table>
          <div class="pagination">
            <el-pagination
              background
              layout="total, prev, pager, next"
              :current-page="productQuery.pageNum"
              :page-size="productQuery.pageSize"
              :total="productPageTotal"
              @current-change="handlePageChangeProduct"
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
            ref="saveProductForm"
            :model="saveProductForm"
            :inline="true"
            :lable-position="labelPosition"
            label-width="120px"
          >
            <el-form-item label="商品选择" prop="productId">
              <el-select
                v-model="saveProductForm.productId"
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
            <el-form-item label="任务金促销价" prop="price">
              <el-input
                v-model="saveProductForm.price"
                placeholder="请输入任务金促销价"
                suffix-icon="“xxxx”"
              ></el-input>
            </el-form-item>
            <el-form-item label="商品划线价格" prop="price">
              <el-input
                v-model="saveProductForm.scribingPrice"
                placeholder="请输入商品划线价格"
                suffix-icon="“xxxx”"
              ></el-input>
            </el-form-item>
            <el-form-item label="活动排序" prop="sOrder">
              <el-input v-model="saveProductForm.sorder" placeholder="请输入排序" suffix-icon="“xxxx”"></el-input>
            </el-form-item>
            <el-form-item label="开始时间" prop="startDate">
              <el-date-picker
                v-model="saveProductForm.startDate"
                type="datetime"
                placeholder="选择开始时间"
              ></el-date-picker>
            </el-form-item>
            <el-form-item label="结束时间" prop="endDate">
              <el-date-picker
                v-model="saveProductForm.endDate"
                type="datetime"
                placeholder="选择结束时间"
              ></el-date-picker>
            </el-form-item>
          </el-form>
          <div class="demo-drawer__footer" style="text-align:center;">
            <el-button type="primary" @click="saveActivityProduct()">确 定</el-button>
            <el-button @click="innerDrawerClose()">取 消</el-button>
          </div>
        </el-drawer>
      </el-drawer>
    </div>
  </div>
</template>
<script>
export default {
  name: "activity",
  data() {
    return {
      productDisabled: false,
      innerDrawerTittle: "",
      innerDrawer: false,
      selectActivityId: "",
      activityProductDrawer: false,
      dialogCentr: true,
      labelPosition: "left",
      query: {
        pageNum: 1,
        pageSize: 10
      },
      productQuery: {
        pageNum: 1,
        pageSize: 5
      },
      tableData: [],
      pageTotal: 0,
      productPageTotal: 0,
      actStatusList: [
        { id: 0, name: "创建" },
        { id: 1, name: "发布" },
        { id: 2, name: "下架" }
      ],
      flagList: [
        { id: 0, name: "需要" },
        { id: 1, name: "不需要" }
      ],
      dialogImageUrl: "",
      imgDialogVisible: false,
      fileLists: [],
      dialogMessage: "",
      saveForm: {},
      saveProductForm: {},
      addOrUpdate: 0,
      productAddOrUpdate: 0,
      editVisible: false,
      ruleValidate: {
        actName: [
          { required: true, message: "请输入活动名称", trigger: "blur" }
        ],
        actType: [
          { required: true, message: "请输入活动类型", trigger: "blur" }
        ],
        actStatus: [
          { required: true, message: "请选择活动状态", trigger: "change" }
        ],
        sOrder: [
          { required: true, message: "请输入活动排序", trigger: "blur" }
        ],
        flag: [
          { required: true, message: "请选择活动倒计时标记", trigger: "change" }
        ]
      },
      activityProductList: [],
      productList: [],
      tableDataLoading: true,
      innerTableDataLoading: true
    };
  },
  created() {
    this.getTableData();
    this.getProductList();
  },
  methods: {
    /* eslint-disable no-unused-vars */
    handleInnerDrawerClose(done) {
      this.$confirm("确定关闭吗？")
        .then(_ => {
          this.resetForm("saveProductForm");
          this.saveProductForm = {};
          done();
        })
        .catch(_ => {});
    },
    handleDrawerClose(done) {
      this.activityProductList = [];
      done();
    },
    innerDrawerClose() {
      this.innerDrawer = false;
      this.resetForm("saveProductForm");
      this.saveProductForm = {};
    },
    //删除活动
    handleDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/activity/deleteActivity";
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
    handleProductDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/activity/deleteActivityProduct";
          var params = { id: row.id };
          this.postRequest(url, params).then(resp => {
            if (resp && resp.status == 200 && resp.data.status == "200") {
              this.$message.success("删除成功");
              this.getActivityProductTableData(this.selectActivityId);
            } else {
              this.$message("删除失败");
            }
          });
        })
        .catch(() => {});
    },
    //保存活动
    saveActivity() {
      this.$refs.saveForm.validate(valid => {
        if (valid) {
          var url = "";
          if (this.addOrUpdate == 0) {
            url = "/activity/insertActivity";
          } else if (this.addOrUpdate == 1) {
            url = "/activity/updateActivity";
          }
          this.saveForm.pictureUrl = this.fileLists[0].url;
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
    saveActivityProduct() {
      var url = "";
      if (this.productAddOrUpdate == 0) {
        url = "/activity/insertActivityProduct";
        this.saveProductForm.actId = this.selectActivityId;
      } else if (this.productAddOrUpdate == 1) {
        url = "/activity/updateActivityProduct";
      }
      var params = this.saveProductForm;
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.$message.success("保存成功");
          this.resetForm("saveProductForm");
          this.saveProductForm = {};
          this.innerDrawer = false;
          this.getActivityProductTableData(this.selectActivityId);
        } else {
          this.$message("保存失败");
        }
      });
    },
    //添加弹出
    handleAdd() {
      this.dialogMessage = "添加活动";
      this.addOrUpdate = 0;
      this.editVisible = true;
    },
    handleProductAdd() {
      this.innerDrawerTittle = "添加活动商品";
      this.productAddOrUpdate = 0;
      this.innerDrawer = true;
      this.productDisabled = false;
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
      var url = "activity/listActivity";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.tableData = resp.data.obj.list;
          this.pageTotal = resp.data.obj.total;
          this.tableDataLoading = false;
        }
      });
    },
    getProductList() {
      var url = "product/listUsableProduct";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.productList = resp.data.obj;
        }
      });
    },
    //编辑弹出
    handleEdit(index, row) {
      this.dialogMessage = "编辑活动";
      this.addOrUpdate = 1;
      this.idx = index;
      this.saveForm = JSON.parse(JSON.stringify(row));
      this.fileLists = [
        {
          name: "",
          url: row.pictureUrl
        }
      ];
      this.editVisible = true;
    },
    handleProductEdit(index, row) {
      this.innerDrawerTittle = "编辑活动商品";
      this.productAddOrUpdate = 1;
      this.productDisabled = true;
      this.saveProductForm = JSON.parse(JSON.stringify(row));
      this.innerDrawer = true;
    },
    //查看活动商品列表
    // eslint-disable-next-line no-unused-vars
    handleMore(index, row) {
      this.selectActivityId = row.id;
      this.activityProductDrawer = true;
      this.innerTableDataLoading = true;
      this.getActivityProductTableData(row.id);
    },
    // eslint-disable-next-line no-unused-vars
    getActivityProductTableData(actId) {
      this.productQuery.id = actId;
      var params = this.productQuery;
      var url = "/activity/listActivityProduct";
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.activityProductList = resp.data.obj.list;
          this.productPageTotal = resp.data.obj.total;
          this.innerTableDataLoading = false;
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
    handlePageChangeProduct(val) {
      this.innerTableDataLoading = true;
      this.$set(this.productQuery, "pageNum", val);
      this.getActivityProductTableData(this.selectActivityId);
    },
    //搜索功能
    handleSearch() {
      this.getTableData();
    },
    handleProductSearch() {
      this.getActivityProductTableData(this.selectActivityId);
    },
    resetQuery() {
      this.query.actName = null;
      this.query.actStatus = null;
      this.query.flag = null;
      this.getTableData();
    },
    resetProductQuery() {
      this.productQuery.productName = null;
      this.getActivityProductTableData(this.selectActivityId);
    },
    // eslint-disable-next-line no-unused-vars
    statusFormat(row, column) {
      if (row.actStatus == 0) {
        return "创建";
      } else if (row.actStatus == 1) {
        return "发布";
      } else if (row.actStatus == 2) {
        return "下架";
      }
    },
    // eslint-disable-next-line no-unused-vars
    flagFormat(row, column) {
      if (row.flag == 0) {
        return "不需要";
      } else if (row.flag == 1) {
        return "需要";
      }
    },
    // eslint-disable-next-line no-unused-vars
    dateFormat(row, column) {
      if (row.createTime === null) {
        return null;
      }
      return this.allDateFormat(row.createTime);
    },
    // eslint-disable-next-line no-unused-vars
    startDateFormat(row, column) {
      if (row.startDate === null) {
        return null;
      }
      return this.allDateFormat(row.startDate);
    },
    // eslint-disable-next-line no-unused-vars
    endDateFormat(row, column) {
      if (row.endDate === null) {
        return null;
      }
      return this.allDateFormat(row.endDate);
    },
    allDateFormat(time) {
      let d = new Date(time); //val 为表格内取到的后台时间
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
.inner-drawer-tittle {
  font-size: 18px;
  color: #303133;
  line-height: 50px;
}
</style>