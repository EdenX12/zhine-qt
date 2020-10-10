<template>
  <div style="margin-top:10px;">
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.bannerName" placeholder="请输入banner名称" class="name-input"></el-input>
        <el-select v-model="query.bannerType" placeholder="请选择banner类别" class="status-select">
          <el-option
            v-for="item in bannerTypeList"
            :key="item.id"
            :label="item.typeName"
            :value="item.id"
          ></el-option>
        </el-select>
        <el-select v-model="query.bannerStatus" placeholder="请选择banner状态" class="status-select">
          <el-option
            v-for="item in bannerStatusList"
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
        <el-table-column label="图片" width="200px" align="center">
          <template slot-scope="scope">
            <el-image
              class="table-banner-img"
              :src="scope.row.bannerUrl"
              :preview-src-list="[scope.row.bannerUrl]"
            ></el-image>
          </template>
        </el-table-column>
        <el-table-column
          prop="bannerName"
          label="banner名称"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column prop="jumpUrl" label="跳转地址" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column
          prop="bannerType"
          label="类型"
          :formatter="typeFormat"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column
          prop="bannerStatus"
          label="状态"
          :formatter="statusFormat"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column prop="sorder" label="排序" align="center" :show-overflow-tooltip="true"></el-table-column>
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
            <el-form-item label="banner名称" prop="bannerName">
              <el-input
                v-model="saveForm.bannerName"
                placeholder="请输入banner名称"
                suffix-icon="“xxxx”"
              ></el-input>
            </el-form-item>
            <el-form-item label="跳转地址" prop="jumpUrl">
              <el-input v-model="saveForm.jumpUrl" placeholder="请输入跳转地址" suffix-icon="“xxxx”"></el-input>
            </el-form-item>
            <el-form-item label="banner排序" prop="sorder">
              <el-input v-model="saveForm.sorder" placeholder="请输入排序" suffix-icon="“xxxx”"></el-input>
            </el-form-item>
            <el-form-item label="banner状态" prop="bannerStatus">
              <el-select v-model="saveForm.bannerStatus" placeholder="请选择状态">
                <el-option
                  v-for="item in bannerStatusList"
                  :key="item.id"
                  :label="item.name"
                  :value="item.id"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="banner类型" prop="bannerType">
              <el-select
                v-model="saveForm.bannerType"
                placeholder="请选择banner类型"
                style="width:100%;"
              >
                <el-option
                  v-for="item in bannerTypeList"
                  :key="item.id"
                  :label="item.typeName"
                  :value="item.id"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="图片上传" prop="bannerUrl" style="width:100%;">
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
            <el-button type="primary" @click="saveBanner()">确定</el-button>
            <el-button @click="editVisible = false">取消</el-button>
          </div>
        </el-dialog>
      </div>
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
      bannerTypeList: [
        { id: 0, typeName: "App引导页" },
        { id: 1, typeName: "首页轮播图" }
      ],
      bannerStatusList: [
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
        bannerName: [
          { required: true, message: "请输入banner名称", trigger: "blur" }
        ],
        bannerType: [
          { required: true, message: "请选择banner类型", trigger: "change" }
        ],
        jumpUrl: [
          { required: true, message: "请输入跳转地址", trigger: "blur" }
        ],
        bannerStatus: [
          { required: true, message: "请选择banner状态", trigger: "change" }
        ],
        sorder: [
          { required: true, message: "请输入banner排序", trigger: "blur" }
        ]
      },
      tableDataLoading: true
    };
  },
  created() {
    this.getTableData();
  },
  methods: {
    //删除banner
    handleDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/banner/deleteBanner";
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
    //保存banner
    saveBanner() {
      this.$refs.saveForm.validate(valid => {
        if (valid) {
          var url = "";
          if (this.addOrUpdate == 0) {
            url = "/banner/insertBanner";
          } else if (this.addOrUpdate == 1) {
            url = "/banner/updateBanner";
          }
          this.saveForm.bannerUrl = this.fileLists[0].url;
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
      this.dialogMessage = "添加banner";
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
      var url = "banner/listBanner";
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
      this.dialogMessage = "编辑banner";
      this.addOrUpdate = 1;
      this.idx = index;
      this.saveForm = JSON.parse(JSON.stringify(row));
      this.fileLists = [
        {
          name: "",
          url: row.bannerUrl
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
    //搜索功能
    handleSearch() {
      this.getTableData();
    },
    resetQuery() {
      this.query.bannerName = null;
      this.query.bannerType = null;
      this.query.bannerStatus = null;
      this.getTableData();
    },
    // eslint-disable-next-line no-unused-vars
    typeFormat(row, column) {
      if (row.bannerType == 0) {
        return "App引导页";
      } else if (row.bannerType == 1) {
        return "首页轮播图";
      }
    },
    // eslint-disable-next-line no-unused-vars
    statusFormat(row, column) {
      if (row.bannerStatus == 0) {
        return "创建";
      } else if (row.bannerStatus == 1) {
        return "发布";
      } else if (row.bannerStatus == 2) {
        return "下架";
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
</style>