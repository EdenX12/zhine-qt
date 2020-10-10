<template>
  <div style="margin-top:10px;">
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.shopName" placeholder="请输入店铺名称" class="name-input"></el-input>
        <el-input v-model="query.legalPerson" placeholder="请输入法人姓名" class="name-input"></el-input>
        <el-input v-model="query.mobile" placeholder="请输入联系手机" class="name-input"></el-input>
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
        <el-table-column label="店铺logo" width="200px" align="center">
          <template slot-scope="scope">
            <el-image
              class="table-banner-img"
              :src="scope.row.shopLogo"
              :preview-src-list="[scope.row.shopLogo]"
            ></el-image>
          </template>
        </el-table-column>
        <el-table-column prop="shopName" label="店铺名称" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column prop="shopStar" label="店铺星级" width="120px" align="center"></el-table-column>
        <el-table-column
          prop="shopScore"
          label="店铺评价分"
          align="center"
          width="120px"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column
          prop="shopService"
          label="店铺评价分"
          align="center"
          width="120px"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column
          prop="legalPerson"
          label="法人代表"
          align="center"
          width="80px"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <el-table-column prop="startDate" label="开店时间" :formatter="startDateFormat" align="center"></el-table-column>
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
        top="50px"
      >
        <el-form
          :rules="ruleValidate"
          ref="saveForm"
          :model="saveForm"
          :inline="true"
          :lable-position="labelPosition"
          label-width="120px"
        >
          <el-form-item label="店铺名称" prop="shopName">
            <el-input v-model="saveForm.shopName" placeholder="请输入店铺名称" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="店铺星级" prop="shopStar">
            <el-input v-model="saveForm.shopStar" placeholder="请输入店铺星级" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="店铺评价分" prop="shopScore">
            <el-input v-model="saveForm.shopScore" placeholder="请输入店铺评价分" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="售后服务分" prop="shopService">
            <el-input v-model="saveForm.shopService" placeholder="请输入售后服务分" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="法人代表" prop="legalPerson">
            <el-input v-model="saveForm.legalPerson" placeholder="请输入法姓名" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="联系手机" prop="mobile">
            <el-input v-model="saveForm.mobile" placeholder="请输入联系手机" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="座机" prop="telphone">
            <el-input v-model="saveForm.telphone" placeholder="请输入座机" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="省" prop="shopProvince">
            <el-input v-model="saveForm.shopProvince" placeholder="请输入省" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="市" prop="shopCity">
            <el-input v-model="saveForm.shopCity" placeholder="请输入市" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="区" prop="shopArea">
            <el-input v-model="saveForm.shopArea" placeholder="请输入区" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="店铺简介" prop="shopDesc">
            <el-input v-model="saveForm.shopDesc" placeholder="请输入座机" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="店铺logo" prop="shopLogo" style="width:100%;">
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
          <el-form-item label="营业执照" prop="license" style="width:100%;">
            <el-upload
              name="file"
              :limit="1"
              action="/admin/upload/file"
              list-type="picture-card"
              :file-list="fileListsRe"
              :on-preview="handlePictureCardPreviewRe"
              :on-success="successRe"
              :on-remove="handleRemoveRe"
              :on-exceed="handleExceed"
            >
              <i class="el-icon-plus"></i>
            </el-upload>
            <el-dialog :visible.sync="imgDialogVisibleRe" size="full" :modal="false" title="查看大图片">
              <img width="60%" :src="dialogImageUrlRe" alt />
            </el-dialog>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="saveShop()">确定</el-button>
          <el-button @click="editVisible = false">取消</el-button>
        </div>
      </el-dialog>
      <!-- 抽屉列表 -->
      <el-drawer
        :visible.sync="couponDrawer"
        direction="rtl"
        size="80%"
        :destroy-on-close="true"
        :withHeader="false"
        :before-close="handleDrawerClose"
      >
        <div style="margin-left:10px;margin-top:10px;">
          <!--搜索-->
          <div class="handle-box">
            <el-button
              type="primary"
              round
              icon="el-icon-circle-plus-outline"
              @click="handleCouponAdd()"
            >添加优惠券</el-button>
          </div>
          <!--数据-->
          <el-table
            :data="couponList"
            :header-cell-style="tableHeaderColor"
            v-loading="innerTableDataLoading"
          >
            <el-table-column label="序号" type="index" width="50" align="center"></el-table-column>
            <el-table-column
              prop="productName"
              label="商品名称"
              align="center"
              :show-overflow-tooltip="true"
              width="110px"
            ></el-table-column>
            <el-table-column prop="couponName" label="优惠券名称" align="center" width="110px"></el-table-column>
            <el-table-column label="券面值" align="center" width="80px">
              <template slot-scope="scope">￥{{scope.row.couponAmount}}</template>
            </el-table-column>
            <el-table-column
              prop="startDate"
              label="券开始日期"
              :formatter="startDateFormat"
              align="center"
            ></el-table-column>min_consume_amount
            <el-table-column prop="endDate" label="券截止日期" :formatter="endDateFormat" align="center"></el-table-column>
            <el-table-column prop="couponQuantity" label="券数量" align="center" width="70px"></el-table-column>
            <el-table-column prop="minConsumeAmount" label="最低消费金额" align="center" width="120px"></el-table-column>
            <el-table-column
              prop="useCon"
              label="使用条件"
              :formatter="useConFormat"
              align="center"
              width="80px"
            ></el-table-column>
            <el-table-column
              prop="couponStatus"
              label="状态"
              :formatter="statusFormat"
              align="center"
              width="80px"
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
          <div class="pagination">
            <el-pagination
              background
              layout="total, prev, pager, next"
              :current-page="couponQuery.pageNum"
              :page-size="couponQuery.pageSize"
              :total="couponPageTotal"
              @current-change="handlePageChangeCoupon"
            ></el-pagination>
          </div>
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
            ref="saveShopCouponForm"
            :model="saveShopCouponForm"
            :inline="true"
            :lable-position="labelPosition"
            label-width="120px"
          >
            <el-form-item label="商品选择" prop="productId">
              <el-select v-model="saveShopCouponForm.productId" filterable placeholder="请选择商品">
                <el-option
                  v-for="item in productList"
                  :key="item.id"
                  :label="item.productName"
                  :value="item.id"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="券名称" prop="couponName">
              <el-input
                v-model="saveShopCouponForm.couponName"
                placeholder="请输入券名称"
                suffix-icon="“xxxx”"
              ></el-input>
            </el-form-item>
            <el-form-item label="券面值" prop="couponAmount">
              <el-input
                v-model="saveShopCouponForm.couponAmount"
                placeholder="请输入券名称"
                suffix-icon="“xxxx”"
              ></el-input>
            </el-form-item>
            <el-form-item label="券数量" prop="couponQuantity">
              <el-input
                v-model="saveShopCouponForm.couponQuantity"
                placeholder="请输入券数量"
                suffix-icon="“xxxx”"
              ></el-input>
            </el-form-item>
            <el-form-item label="单人领取限制" prop="perLimit">
              <el-input
                v-model="saveShopCouponForm.perLimit"
                placeholder="请输入单人领取限制"
                suffix-icon="“xxxx”"
              ></el-input>
            </el-form-item>
            <el-form-item label="最低消费金额" prop="minConsumeAmount">
              <el-input
                v-model="saveShopCouponForm.minConsumeAmount"
                placeholder="请输入最低消费金额"
                suffix-icon="“xxxx”"
              ></el-input>
            </el-form-item>
            <el-form-item label="使用条件" prop="useCon">
              <el-select v-model="saveShopCouponForm.useCon" placeholder="请选择状态">
                <el-option
                  v-for="item in UseConList"
                  :key="item.id"
                  :label="item.name"
                  :value="item.id"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="状态" prop="couponStatus">
              <el-select v-model="saveShopCouponForm.couponStatus" placeholder="请选择商品状态">
                <el-option
                  v-for="item in couponStatusList"
                  :key="item.id"
                  :label="item.name"
                  :value="item.id"
                ></el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="开始时间" prop="startDate">
              <el-date-picker
                v-model="saveShopCouponForm.startDate"
                type="datetime"
                placeholder="选择开始时间"
              ></el-date-picker>
            </el-form-item>
            <el-form-item label="结束时间" prop="endDate">
              <el-date-picker
                v-model="saveShopCouponForm.endDate"
                type="datetime"
                placeholder="选择结束时间"
              ></el-date-picker>
            </el-form-item>
          </el-form>
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
  name: "shopManage",
  data() {
    return {
      UseConList: [
        { id: 0, name: "立减" },
        { id: 1, name: "满减" },
        { id: 2, name: "超级券" }
      ],
      couponStatusList: [
        { id: 0, name: "创建" },
        { id: 1, name: "发布" },
        { id: 2, name: "下架" }
      ],
      dialogCentr: true,
      labelPosition: "right",
      query: {
        pageNum: 1,
        pageSize: 10
      },
      couponQuery: {
        pageNum: 1,
        pageSize: 5
      },
      tableData: [],
      pageTotal: 0,
      couponPageTotal: 0,
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
      dialogImageUrlRe: "",
      imgDialogVisible: false,
      imgDialogVisibleRe: false,
      fileLists: [],
      fileListsRe: [],
      dialogMessage: "",
      saveForm: {},
      addOrUpdate: 0,
      editVisible: false,
      ruleValidate: {
        couponName: [
          { required: true, message: "请输入优惠券名称", trigger: "blur" }
        ]
      },
      tableDataLoading: true,
      couponList: [],
      innerTableDataLoading: true,
      couponDrawer: false,
      selectShopId: "",
      saveShopCouponForm: {},
      innerDrawer: false,
      innerDrawerTittle: "",
      shopCouponAddOrUpdate: 0,
      productList: []
    };
  },
  created() {
    this.getTableData();
    this.getProductList();
  },
  methods: {
    //关闭抽屉
    handleDrawerClose(done) {
      this.couponList = [];
      this.couponQuery.pageNum = 1;
      done();
    },
    /* eslint-disable no-unused-vars */
    handleInnerDrawerClose(done) {
      this.$confirm("确定关闭吗？")
        .then(_ => {
          this.resetForm("saveShopCouponForm");
          this.saveShopCouponForm = {};
          done();
        })
        .catch(_ => {});
    },
    innerDrawerClose() {
      this.innerDrawer = false;
      this.resetForm("saveShopCouponForm");
      this.saveShopCouponForm = {};
    },
    //查看商户优惠券列表
    // eslint-disable-next-line no-unused-vars
    handleMore(index, row) {
      this.selectShopId = row.id;
      this.couponDrawer = true;
      this.innerTableDataLoading = true;
      this.getCouponTableData(row.id);
    },
    //商户优惠券列表数据
    getCouponTableData(id) {
      var url = "/coupon/listShopCoupon";
      this.couponQuery.id = id;
      this.postRequest(url, this.couponQuery).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.couponList = resp.data.obj.list;
          this.couponPageTotal = resp.data.obj.total;
          this.innerTableDataLoading = false;
        }
      });
    },
    //删除商户
    handleDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/shop/deleteShop";
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
    handleCouponDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/coupon/deleteShopCoupon";
          var params = { id: row.id };
          this.postRequest(url, params).then(resp => {
            if (resp && resp.status == 200 && resp.data.status == "200") {
              this.$message.success("删除成功");
              this.getCouponTableData(this.selectShopId);
            } else {
              this.$message("删除失败");
            }
          });
        })
        .catch(() => {});
    },
    //保存商户
    saveShop() {
      this.$refs.saveForm.validate(valid => {
        if (valid) {
          var url = "";
          if (this.addOrUpdate == 0) {
            url = "/shop/insertShop";
          } else if (this.addOrUpdate == 1) {
            url = "/shop/updateShop";
          }
          this.saveForm.shopLogo = this.fileLists[0].url;
          this.saveForm.license = this.fileListsRe[0].url;
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
    saveShopCoupon() {
      var url = "";
      if (this.shopCouponAddOrUpdate == 0) {
        url = "/coupon/insertShopCoupon";
        this.saveShopCouponForm.shopId = this.selectShopId;
      } else if (this.shopCouponAddOrUpdate == 1) {
        url = "/coupon/updateShopCoupon";
      }
      var params = this.saveShopCouponForm;
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.$message.success("保存成功");
          this.resetForm("saveShopCouponForm");
          this.saveShopCouponForm = {};
          this.innerDrawer = false;
          this.getCouponTableData(this.selectShopId);
        } else {
          this.$message("保存失败");
        }
      });
    },
    //添加弹出
    handleAdd() {
      this.dialogMessage = "添加店铺";
      this.addOrUpdate = 0;
      this.editVisible = true;
    },
    handleCouponAdd() {
      this.innerDrawerTittle = "添加店铺优惠券";
      this.shopCouponAddOrUpdate = 0;
      this.innerDrawer = true;
    },
    //编辑弹出
    handleEdit(index, row) {
      this.dialogMessage = "编辑店铺";
      this.addOrUpdate = 1;
      this.idx = index;
      this.saveForm = JSON.parse(JSON.stringify(row));
      this.fileLists = [
        {
          name: "",
          url: row.shopLogo
        }
      ];
      this.fileListsRe = [
        {
          name: "",
          url: row.license
        }
      ];
      this.editVisible = true;
    },
    handleCouponEdit(index, row) {
      this.innerDrawerTittle = "编辑商铺优惠券";
      this.shopCouponAddOrUpdate = 1;
      this.saveShopCouponForm = JSON.parse(JSON.stringify(row));
      this.innerDrawer = true;
    },
    //关闭弹窗
    closeDialog() {
      this.resetForm("saveForm");
      this.saveForm = {};
      this.fileLists = [];
      this.fileListsRe = [];
    },
    resetForm(formName) {
      this.$refs[formName].clearValidate();
    },
    getTableData() {
      var url = "/shop/listShop";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.tableData = resp.data.obj.list;
          this.pageTotal = resp.data.obj.total;
          this.tableDataLoading = false;
        }
      });
    },
    getProductList() {
      var url = "/product/listUsableProduct";
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.productList = resp.data.obj;
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
    handlePageChangeCoupon(val) {
      this.innerTableDataLoading = true;
      this.$set(this.couponQuery, "pageNum", val);
      this.getCouponTableData(this.selectShopId);
    },
    //搜索功能
    handleSearch() {
      this.getTableData();
    },
    resetQuery() {
      this.query.shopName = null;
      this.query.legalPerson = null;
      this.query.mobile = null;
      this.getTableData();
    },
    // eslint-disable-next-line no-unused-vars
    statusFormat(row, column) {
      if (row.couponStatus == 0) {
        return "创建";
      } else if (row.couponStatus == 1) {
        return "发布";
      } else if (row.couponStatus == 2) {
        return "下架";
      }
    },
    // eslint-disable-next-line no-unused-vars
    startDateFormat(row, column) {
      return this.dateFormat(row.startDate);
    },
    // eslint-disable-next-line no-unused-vars
    endDateFormat(row, column) {
      return this.dateFormat(row.endDate);
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
    // eslint-disable-next-line no-unused-vars
    handleRemoveRe(file, fileList) {
      this.fileListsRe = [];
    },
    //查看大图
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.imgDialogVisible = true;
    },
    handlePictureCardPreviewRe(file) {
      this.dialogImageUrlRe = file.url;
      this.imgDialogVisibleRe = true;
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
    successRe(response, file, fileList) {
      this.fileListsRe = [
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
    useConFormat(row, column) {
      if (row.useCon == 0) {
        return "立减";
      } else if (row.useCon == 1) {
        return "满减";
      } else if (row.useCon == 2) {
        return "超级券";
      }
    }
  }
};
</script>
<style scoped>
.table-banner-img {
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