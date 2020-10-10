<template>
  <div style="margin-top:10px;">
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.valueName" placeholder="请输入规格值名称" class="name-input"></el-input>
        <el-input v-model="query.valueType" placeholder="请输入规格值类型" class="name-input"></el-input>
        <el-select v-model="query.productTypeId" placeholder="请选择分类类别" class="status-select">
          <el-option
            v-for="item in categoryList"
            :key="item.id"
            :label="item.typeName"
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
        <el-table-column label="图片" width="100px" align="center">
          <template slot-scope="scope">
            <el-image
              class="table-spec-value-img"
              :src="scope.row.valueImage"
              :preview-src-list="[scope.row.valueImage]"
            ></el-image>
          </template>
        </el-table-column>
        <el-table-column
          prop="productTypeName"
          label="类别"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column
          prop="valueName"
          label="规格值名称"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column
          prop="valueType"
          label="规格值类型"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column prop="valueSort" label="排序" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column prop="createTime" label="创建时间" :formatter="dateFormat" align="center"></el-table-column>
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
          <el-form-item label="分类类型" prop="productTypeId">
            <el-select v-model="saveForm.productTypeId" placeholder="请选择分类类型" style="width:100%;">
              <el-option
                v-for="item in categoryList"
                :key="item.id"
                :label="item.typeName"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="规格值名称" prop="valueName">
            <el-input v-model="saveForm.valueName" placeholder="请输入规格值名称" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="规格值类型" prop="valueType">
            <el-input v-model="saveForm.valueType" placeholder="请输入规格值类型" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="规格值排序" prop="valueSort">
            <el-input v-model="saveForm.valueSort" placeholder="请输入排序" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="图片上传" prop="valueImage" style="width:100%;">
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
          <el-button type="primary" @click="saveProductSpecValue()">确定</el-button>
          <el-button @click="editVisible = false">取消</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>
<script>
export default {
  name: "productSpecValue",
  data() {
    return {
      query: {
        pageNum: 1,
        pageSize: 10
      },
      labelPosition: "left",
      tableData: [],
      pageTotal: 0,
      saveForm: {},
      dialogImageUrl: "",
      imgDialogVisible: false,
      fileLists: [],
      dialogMessage: "",
      addOrUpdate: 0,
      editVisible: false,
      categoryList: [],
      dialogCentr: true,
      ruleValidate: {
        valueName: [
          { required: true, message: "请输入规格值名称", trigger: "blur" }
        ]
      },
      tableDataLoading: true
    };
  },
  created() {
    this.getTableData();
    this.getCategoryList();
  },
  methods: {
    getTableData() {
      var url = "/product/listProductSpecValuePage";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.tableData = resp.data.obj.list;
          this.pageTotal = resp.data.obj.total;
          this.tableDataLoading = false;
        }
      });
    },
    //保存规格值
    saveProductSpecValue() {
      this.$refs.saveForm.validate(valid => {
        if (valid) {
          var url = "";
          if (this.addOrUpdate == 0) {
            url = "/product/insertProductSpecValue";
          } else if (this.addOrUpdate == 1) {
            url = "/product/updateProductSpecValue";
          }
          if (this.fileLists.length == 0) {
            this.saveForm.valueImage = null;
          } else {
            this.saveForm.valueImage = this.fileLists[0].url;
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
    //商品分类列表
    getCategoryList() {
      var url = "/product/listCategory";
      var params = {};
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.categoryList = resp.data.obj;
        }
      });
    },
    //添加弹出
    handleAdd() {
      this.dialogMessage = "添加规格值";
      this.addOrUpdate = 0;
      this.editVisible = true;
    },
    //编辑弹出
    handleEdit(index, row) {
      this.dialogMessage = "编辑规格值";
      this.addOrUpdate = 1;
      this.idx = index;
      this.saveForm = JSON.parse(JSON.stringify(row));
      this.fileLists = [
        {
          name: "",
          url: row.valueImage
        }
      ];
      this.saveForm.productTypeId = parseInt(row.productTypeId);
      this.editVisible = true;
    },
    //删除规格值
    handleDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/product/deleteProductSpecValue";
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
    //搜索功能
    handleSearch() {
      this.tableDataLoading = true;
      this.query.pageNum = 1;
      this.getTableData();
    },
    //重置
    resetQuery() {
      this.query.valueName = null;
      this.query.valueType = null;
      this.query.productTypeId = null;
      this.getTableData();
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