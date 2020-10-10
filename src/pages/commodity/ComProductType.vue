<template>
  <div style="margin-top:10px;">
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.typeName" placeholder="请输入分类名称" class="name-input"></el-input>
        <el-select v-model="query.productStatus" placeholder="请选择状态" class="status-select">
          <el-option v-for="item in statusList" :key="item.id" :label="item.name" :value="item.id"></el-option>
        </el-select>
        <el-select v-model="query.flag" placeholder="请选择推荐" class="status-select">
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
        <el-table-column label="图片" width="100px" align="center">
          <template slot-scope="scope">
            <el-image
              class="table-spec-value-img"
              :src="scope.row.typeImg"
              :preview-src-list="[scope.row.typeImg]"
            ></el-image>
          </template>
        </el-table-column>
        <el-table-column prop="typeName" label="分类名称" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column
          prop="typeStatus"
          label="状态"
          :formatter="statusFormat"
          align="center"
          width="100px"
        ></el-table-column>
        <el-table-column
          prop="flag"
          label="是否推荐"
          :formatter="flagFormat"
          align="center"
          width="100px"
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
          <el-form-item label="分类名称" prop="typeName">
            <el-input v-model="saveForm.typeName" placeholder="请输入分类名称" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="分类状态" prop="productTypeId">
            <el-select v-model="saveForm.typeStatus" placeholder="请选择分类状态" style="width:100%;">
              <el-option
                v-for="item in statusList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="是否推荐" prop="flag">
            <el-select v-model="saveForm.flag" placeholder="请选择分类状态" style="width:100%;">
              <el-option
                v-for="item in flagList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="分类排序" prop="sOrder">
            <el-input v-model="saveForm.sOrder" placeholder="请输入分类排序" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="图片上传" prop="typeImg" style="width:100%;">
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
          <el-button type="primary" @click="saveProductType()">确定</el-button>
          <el-button @click="editVisible = false">取消</el-button>
        </div>
      </el-dialog>
      <!-- 抽屉 -->
      <el-drawer
        :visible.sync="productTypeDrawer"
        direction="rtl"
        size="70%"
        :destroy-on-close="true"
        :withHeader="false"
        :before-close="handleDrawerClose"
      >
        <div style="margin-left:10px;margin-top:10px;">
          <div class="handle-box">
            <el-input v-model="childQuery.typeName" placeholder="请输入分类名称" class="name-input"></el-input>
            <el-select v-model="childQuery.productStatus" placeholder="请选择状态" class="status-select">
              <el-option
                v-for="item in statusList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
            <el-select v-model="childQuery.flag" placeholder="请选择推荐" class="status-select">
              <el-option
                v-for="item in flagList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
            <el-button type="primary" round icon="el-icon-search" @click="innerHandleSearch()">搜索</el-button>
            <el-button type="danger" round icon="el-icon-refresh" @click="innerResetQuery()">重置</el-button>
            <el-button
              type="primary"
              round
              icon="el-icon-circle-plus-outline"
              @click="handleInnerAdd()"
            >添加</el-button>
          </div>
          <el-table
            :data="productTypeChildData"
            :header-cell-style="tableHeaderColor"
            class="table"
            ref="multipleTable"
            header-cell-class-name="table-header"
            v-loading="innerTableDataLoading"
          >
            <el-table-column label="序号" type="index" width="50" align="center"></el-table-column>
            <el-table-column label="图片" width="100px" align="center">
              <template slot-scope="scope">
                <el-image
                  class="table-spec-value-img"
                  :src="scope.row.typeImg"
                  :preview-src-list="[scope.row.typeImg]"
                ></el-image>
              </template>
            </el-table-column>
            <el-table-column
              prop="typeName"
              label="分类名称"
              align="center"
              :show-overflow-tooltip="true"
            ></el-table-column>
            <el-table-column
              prop="typeStatus"
              label="状态"
              :formatter="statusFormat"
              align="center"
              width="100px"
            ></el-table-column>
            <el-table-column
              prop="flag"
              label="是否推荐"
              :formatter="flagFormat"
              align="center"
              width="100px"
            ></el-table-column>
            <el-table-column prop="sOrder" label="排序" align="center" :show-overflow-tooltip="true"></el-table-column>
            <el-table-column prop="createTime" label="创建时间" :formatter="dateFormat" align="center"></el-table-column>
            <el-table-column label="操作" align="center">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  circle
                  @click="innerHandleEdit(scope.$index, scope.row)"
                ></el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  circle
                  @click="innerHandleDelete(scope.$index, scope.row)"
                ></el-button>
              </template>
            </el-table-column>
          </el-table>
          <div class="pagination">
            <el-pagination
              background
              layout="total, prev, pager, next"
              :current-page="childQuery.pageNum"
              :page-size="childQuery.pageSize"
              :total="childPageTotal"
              @current-change="handlePageChangeChild"
            ></el-pagination>
          </div>
        </div>
      </el-drawer>
    </div>
  </div>
</template>
<script>
export default {
  name: "productType",
  data() {
    return {
      query: {
        pageNum: 1,
        pageSize: 10
      },
      childQuery: {
        pageNum: 1,
        pageSize: 5
      },
      labelPosition: "left",
      tableData: [],
      pageTotal: 0,
      childPageTotal: 0,
      saveForm: {},
      dialogImageUrl: "",
      imgDialogVisible: false,
      fileLists: [],
      dialogMessage: "",
      addOrUpdate: 0,
      parentOrChild: 0,
      editVisible: false,
      categoryList: [],
      dialogCentr: true,
      ruleValidate: {
        typeName: [
          { required: true, message: "请输入分类名称", trigger: "blur" }
        ]
      },
      statusList: [
        { id: 0, name: "不可用" },
        { id: 1, name: "可用" }
      ],
      flagList: [
        { id: 0, name: "普通" },
        { id: 1, name: "推荐" }
      ],
      productTypeDrawer: false,
      selectProductTypeId: "",
      productTypeChildData: [],
      tableDataLoading: true,
      innerTableDataLoading: true
    };
  },
  created() {
    this.getTableData();
  },
  methods: {
    getTableData() {
      var url = "/product/listProductType";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.tableData = resp.data.obj.list;
          this.pageTotal = resp.data.obj.total;
          this.tableDataLoading = false;
        }
      });
    },
    //获取二级分类
    getProductTypeChild(parentTypeId) {
      var url = "/product/listProductTypeChild";
      this.childQuery.parentId = parentTypeId;
      this.postRequest(url, this.childQuery).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.productTypeChildData = resp.data.obj.list;
          this.childPageTotal = resp.data.obj.total;
          this.innerTableDataLoading = false;
        }
      });
    },
    //保存分类
    saveProductType() {
      this.$refs.saveForm.validate(valid => {
        if (valid) {
          var url = "";
          if (this.addOrUpdate == 0) {
            url = "/product/insertProductType";
          } else if (this.addOrUpdate == 1) {
            url = "/product/updateProductType";
          }
          if (this.fileLists.length == 0) {
            this.saveForm.typeImg = null;
          } else {
            this.saveForm.typeImg = this.fileLists[0].url;
          }
          var params = this.saveForm;
          this.postRequest(url, params).then(resp => {
            if (resp && resp.status == 200 && resp.data.status == "200") {
              this.$message.success("保存成功");
              this.editVisible = false;
              if (this.parentOrChild == 0) {
                this.getTableData();
              } else if (this.parentOrChild == 1) {
                this.getProductTypeChild(this.selectProductTypeId);
              }
            } else {
              this.$message("保存失败");
            }
          });
        }
      });
    },
    // eslint-disable-next-line no-unused-vars
    handleMore(index, row) {
      this.selectProductTypeId = row.id;
      this.productTypeDrawer = true;
      this.innerTableDataLoading = true;
      this.getProductTypeChild(row.id);
    },
    //添加弹出
    handleAdd() {
      this.dialogMessage = "添加分类";
      this.addOrUpdate = 0;
      this.editVisible = true;
      this.saveForm.level = 1;
      this.parentOrChild = 0;
    },
    handleInnerAdd() {
      this.dialogMessage = "添加分类";
      this.addOrUpdate = 0;
      this.editVisible = true;
      this.saveForm.level = 2;
      this.parentOrChild = 1;
      this.saveForm.parentId = this.selectProductTypeId;
    },
    //编辑弹出
    handleEdit(index, row) {
      this.dialogMessage = "编辑分类";
      this.addOrUpdate = 1;
      this.idx = index;
      this.saveForm = JSON.parse(JSON.stringify(row));
      this.fileLists = [
        {
          name: "",
          url: row.typeImg
        }
      ];
      this.saveForm.productTypeId = parseInt(row.productTypeId);
      this.editVisible = true;
      this.parentOrChild = 0;
    },
    innerHandleEdit(index, row) {
      this.dialogMessage = "编辑分类";
      this.addOrUpdate = 1;
      this.idx = index;
      this.saveForm = JSON.parse(JSON.stringify(row));
      this.fileLists = [
        {
          name: "",
          url: row.typeImg
        }
      ];
      this.saveForm.productTypeId = parseInt(row.productTypeId);
      this.editVisible = true;
      this.parentOrChild = 1;
    },
    //删除分类
    handleDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/product/deleteProductType";
          var params = { id: row.id };
          this.postRequest(url, params).then(resp => {
            if (resp && resp.status == 200 && resp.data.status == "200") {
              this.$message.success("删除成功");
              this.getTableData();
            }
          });
        })
        .catch(() => {});
    },
    innerHandleDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/product/deleteProductType";
          var params = { id: row.id };
          this.postRequest(url, params).then(resp => {
            if (resp && resp.status == 200 && resp.data.status == "200") {
              this.$message.success("删除成功");
              this.getProductTypeChild(this.selectProductTypeId);
            } else {
              this.$message("删除失败");
            }
          });
        })
        .catch(() => {});
    },
    // eslint-disable-next-line no-unused-vars
    statusFormat(row, column) {
      if (row.typeStatus == 0) {
        return "不可用";
      } else if (row.typeStatus == 1) {
        return "可用";
      }
    },

    // eslint-disable-next-line no-unused-vars
    flagFormat(row, column) {
      if (row.flag == 0) {
        return "普通";
      }
      if (row.flag == 1) {
        return "推荐";
      }
    },
    //关闭抽屉
    handleDrawerClose(done) {
      this.productTypeChildData = [];
      this.childQuery.pageNum = 1;
      done();
    },
    // eslint-disable-next-line no-unused-vars
    tableHeaderColor(row, rowIndex) {
      return "background-color:#EBEEF5;";
    },
    //分页
    handlePageChange(val) {
      this.tableDataLoading = true;
      this.$set(this.query, "pageNum", val);
      this.getTableData(this.query);
    },
    handlePageChangeChild(val) {
      this.innerTableDataLoading = true;
      this.$set(this.childQuery, "pageNum", val);
      this.getProductTypeChild(this.selectProductTypeId);
    },
    //搜索功能
    handleSearch() {
      this.tableDataLoading = true;
      this.query.pageNum = 1;
      this.getTableData();
    },
    innerHandleSearch() {
      this.innerTableDataLoading = true;
      this.childQuery.pageNum = 1;
      this.getProductTypeChild(this.selectProductTypeId);
    },
    //重置
    resetQuery() {
      this.query.typeName = null;
      this.query.typeStatus = null;
      this.query.flag = null;
      this.getTableData();
    },
    innerResetQuery() {
      this.childQuery.typeName = null;
      this.childQuery.typeStatus = null;
      this.childQuery.flag = null;
      this.getProductTypeChild(this.selectProductTypeId);
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
    }
  }
};
</script>
<style scoped>
.table-spec-value-img {
  display: block;
  margin: auto;
  width: 50px;
  height: 50px;
}
.name-input {
  width: 200px;
  margin-right: 10px;
}
.status-select {
  margin-right: 10px;
}
.handle-box {
  margin-bottom: 10px;
  text-align: left;
}
</style>