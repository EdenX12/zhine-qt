<template>
  <div>
    <div class="container">
      <!--搜索条件-->
      <div class="handle-box">
        <el-input v-model="query.productName" placeholder="商品名称" class="name-input"></el-input>
        <el-select
          v-model="query.shopId"
          placeholder="请选择商户"
          class="type-select"
          :disabled="shopDisabled"
        >
          <el-option
            v-for="item in shopList"
            :key="item.id"
            :label="item.shopName"
            :value="item.id"
          ></el-option>
        </el-select>
        <el-cascader
          :props="selectProps"
          v-model="query.typeId"
          placeholder="请选择商品类别"
          class="type-select"
        ></el-cascader>
        <el-select v-model="query.productStatus" placeholder="请选择商品状态" class="type-select">
          <el-option
            v-for="item in productStatusList"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          ></el-option>
        </el-select>
        <el-select v-model="query.productType" placeholder="请选择产品类型" class="type-select">
          <el-option
            v-for="item in productTypeList"
            :key="item.id"
            :label="item.name"
            :value="item.id"
          ></el-option>
        </el-select>
        <el-button
          class="query-button"
          type="primary"
          round
          icon="el-icon-search"
          @click="handleSearch()"
        >搜索</el-button>
        <el-button
          class="query-button"
          type="danger"
          round
          icon="el-icon-refresh"
          @click="resetQuery()"
        >重置</el-button>
        <el-button
          class="query-button"
          style="margin-right: 10px;"
          type="primary"
          round
          icon="el-icon-circle-plus-outline"
          @click="handleAdd()"
        >添加</el-button>
        <el-select
          v-model="selectRecommendType"
          placeholder="请选择推荐分类"
          class="type-select"
          :disabled="recommendTypeDisabled"
        >
          <el-option
            v-for="item in recommendTypeData"
            :key="item.id"
            :label="item.recommendTitle"
            :value="item.id"
          ></el-option>
        </el-select>
        <el-button
          style="margin-right: 10px;"
          class="query-button"
          type="primary"
          round
          icon="el-icon-circle-plus-outline"
          @click="handleAddToRecommend()"
        >添加到推荐</el-button>
        <el-select
          v-model="selectActivity"
          placeholder="请选择活动"
          class="type-select"
          :disabled="activityDisabled"
        >
          <el-option
            v-for="item in activityData"
            :key="item.id"
            :label="item.actName"
            :value="item.id"
          ></el-option>
        </el-select>
        <el-button
          class="query-button"
          type="primary"
          round
          icon="el-icon-circle-plus-outline"
          @click="handleAddToActivity()"
        >添加到活动</el-button>
      </div>
      <!--数据表单-->
      <el-table
        :data="tableData"
        :header-cell-style="tableHeaderColor"
        class="table"
        ref="multipleTable"
        header-cell-class-name="table-header"
        v-loading="tableDataLoading"
        @selection-change="handleSelectionChange"
      >
        <el-table-column type="selection" width="50"></el-table-column>
        <el-table-column label="序号" type="index" width="50" align="center"></el-table-column>
        <el-table-column label="商品图片" width="80px" align="center">
          <template slot-scope="scope">
            <el-image
              class="table-product-img"
              :src="scope.row.productImg"
              :preview-src-list="[scope.row.productImg]"
            ></el-image>
          </template>
        </el-table-column>
        <el-table-column prop="shopName" label="商户名称" align="center" :show-overflow-tooltip="true"></el-table-column>
        <el-table-column
          prop="productName"
          label="商品名称"
          align="center"
          :show-overflow-tooltip="true"
        ></el-table-column>
        <!-- <el-table-column label="价格(元)" align="center" width="80px">
          <template slot-scope="scope">￥{{scope.row.productPrice}}</template>
        </el-table-column>-->
        <el-table-column prop="totalNumber" label="数量(任务线总数)" align="center" width="130px"></el-table-column>
        <el-table-column prop="taskNumber" label="任务份数" align="center" width="80px"></el-table-column>
        <el-table-column
          prop="productStatus"
          label="状态"
          :formatter="statusFormat"
          align="center"
          width="70px"
        ></el-table-column>
        <el-table-column
          prop="productType"
          label="产品类型"
          :formatter="typeFormat"
          align="center"
          width="80px"
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
            <!-- <el-button
              type="danger"
              icon="el-icon-delete"
              circle
              @click="handleDelete(scope.$index, scope.row)"
            ></el-button> -->
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
        :before-close="handleClose"
        top="0"
        width="60%"
      >
        <el-form
          ref="saveForm"
          :rules="ruleValidate"
          :model="saveForm"
          :inline="true"
          label-width="130px"
          :lable-position="labelPosition"
        >
          <el-form-item label="商品名称" prop="productName">
            <el-input v-model="saveForm.productName" placeholder="请输入商品名称" suffix-icon="“xxxx”"></el-input>
          </el-form-item>
          <el-form-item label="所属分类" prop="typeId">
            <el-cascader :props="props" v-model="saveForm.typeId" placeholder="请选择商品类别"></el-cascader>
          </el-form-item>
          <el-form-item label="价格(元)" prop="productPrice">
            <el-input
              v-model="saveForm.productPrice"
              placeholder="请输入价格"
              onkeyup="this.value=this.value.replace(/[^\d.]/g,'');"
              suffix-icon="“xxxx”"
            ></el-input>
          </el-form-item>
          <el-form-item label="总佣金(元)" prop="totalReward">
            <el-input
              v-model="saveForm.totalReward"
              :disabled="totalRewardDisabled"
              placeholder="请输入佣金"
              suffix-icon="“xxxx”"
            ></el-input>
          </el-form-item>
          <el-form-item label="任务单价(元)" prop="taskPrice">
            <el-input
              v-model="saveForm.taskPrice"
              :disabled="taskPriceDisabled"
              placeholder="请输入任务金"
              suffix-icon="“xxxx”"
            ></el-input>
          </el-form-item>
          <el-form-item label="任务份数" prop="taskNumber">
            <el-input
              v-model="saveForm.taskNumber"
              :disabled="taskNumberDisabled"
              placeholder="请输入任务份数"
              suffix-icon="“xxxx”"
            ></el-input>
          </el-form-item>
          <el-form-item label="状态" prop="productStatus">
            <el-select v-model="saveForm.productStatus" placeholder="请选择商品状态">
              <el-option
                v-for="item in productStatusList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="产品类型" prop="productType">
            <el-select v-model="saveForm.productType" placeholder="请选择产品类型">
              <el-option
                v-for="item in productTypeList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="商户" prop="shopId">
            <el-select v-model="saveForm.shopId" placeholder="请选择商户" :disabled="editShopDisabled">
              <el-option
                v-for="item in shopList"
                :key="item.id"
                :label="item.shopName"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="长方形图" prop="homeProductImg" style="width:100%;">
            <el-upload
              name="file"
              action="/admin/upload/file"
              list-type="picture-card"
              :limit="1"
              :file-list="homeFileLists"
              :on-preview="handlePictureCardPreview"
              :on-success="homeSuccess"
              :on-remove="handleHomeRemove"
              :on-exceed="handleExceed"
            >
              <i class="el-icon-plus"></i>
            </el-upload>
            <el-dialog :visible.sync="imgDialogVisible" size="full" :modal="false" title="查看大图片">
              <img width="100%" :src="dialogImageUrl" alt />
            </el-dialog>
          </el-form-item>
          <el-form-item label="主图上传" prop="productImg" style="width:100%;">
            <el-upload
              name="file"
              action="/admin/upload/file"
              list-type="picture-card"
              :limit="1"
              :file-list="fileLists"
              :on-preview="handlePictureCardPreview"
              :on-success="success"
              :on-remove="handleRemove"
              :on-exceed="handleExceed"
            >
              <i class="el-icon-plus"></i>
            </el-upload>
            <el-dialog :visible.sync="imgDialogVisible" size="full" :modal="false" title="查看大图片">
              <img width="100%" :src="dialogImageUrl" alt />
            </el-dialog>
          </el-form-item>
          <el-form-item label="缩略图上传" prop="detailProductImg" style="width:100%;">
            <el-upload
              name="file"
              action="/admin/upload/file"
              list-type="picture-card"
              :file-list="detailFileLists"
              :on-preview="handlePictureCardPreview"
              :on-success="detailSuccess"
              :on-remove="handleDetailRemove"
            >
              <i class="el-icon-plus"></i>
            </el-upload>
            <el-dialog :visible.sync="imgDialogVisible" size="full" :modal="false" title="查看大图片">
              <img width="100%" :src="dialogImageUrl" alt />
            </el-dialog>
          </el-form-item>
          <el-form-item label="商品详情" prop="productDes">
            <quill-editor
              v-model="saveForm.productDetail"
              :options="quillOption"
              style="height: 500px;width:600px;"
            ></quill-editor>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="saveProduct()" :loading="saveLoading">确定</el-button>
          <el-button @click="editVisible = false">取消</el-button>
        </div>
      </el-dialog>
      <!-- 抽屉 -->
      <el-drawer
        :visible.sync="productSpecDrawer"
        direction="rtl"
        size="70%"
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
              @click="handleProductSpecAdd()"
            >添加</el-button>
          </div>
          <!--数据-->
          <el-table
            :data="productSpecList"
            :header-cell-style="tableHeaderColor"
            v-loading="innerTableDataLoading"
          >
            <el-table-column label="序号" type="index" width="50" align="center"></el-table-column>
            <el-table-column
              prop="productSpecValueType"
              label="规格值类型"
              align="center"
              :show-overflow-tooltip="true"
              width="110px"
            ></el-table-column>
            <el-table-column
              prop="productSpecValueName"
              label="规格值名称"
              align="center"
              :show-overflow-tooltip="true"
            ></el-table-column>
            <el-table-column label="商品数量" prop="productNumber" align="center"></el-table-column>
            <el-table-column label="商品库存" prop="stockNumber" align="center"></el-table-column>
            <el-table-column label="商品价格" align="center">
              <template slot-scope="scope">￥{{scope.row.productPrice}}</template>
            </el-table-column>
            <el-table-column label="划线价格" align="center">
              <template slot-scope="scope">￥{{scope.row.scribingPrice}}</template>
            </el-table-column>
            <el-table-column label="成本价格" align="center">
              <template slot-scope="scope">￥{{scope.row.costPrice}}</template>
            </el-table-column>
            <el-table-column prop="createTime" label="创建时间" :formatter="dateFormat" align="center"></el-table-column>
            <el-table-column label="操作" align="center">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  circle
                  @click="handleProductSpecEdit(scope.$index, scope.row)"
                ></el-button>
                <!-- <el-button
                  type="danger"
                  icon="el-icon-delete"
                  circle
                  @click="handleProductSpecDelete(scope.$index, scope.row)"
                ></el-button>-->
              </template>
            </el-table-column>
          </el-table>
          <!-- 嵌套抽屉 -->
          <el-drawer
            size="60%"
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
              :rules="specRuleValidate"
              ref="saveProductSpecForm"
              :model="saveProductSpecForm"
              :inline="true"
              :lable-position="labelPosition"
              label-width="120px"
            >
              <el-form-item style="margin-left:80px;">
                <template>
                  <el-transfer
                    v-model="selectProductSpecValueList"
                    :data="productSpecValueList"
                    target-order="push"
                    :titles="transferTittle"
                  ></el-transfer>
                </template>
              </el-form-item>
              <el-form-item label="商品数量" prop="productNumber">
                <el-input v-model="saveProductSpecForm.productNumber" placeholder="请输入商品数量"></el-input>
              </el-form-item>
              <el-form-item label="商品价格" prop="productPrice">
                <el-input v-model="saveProductSpecForm.productPrice" placeholder="请输入商品价格"></el-input>
              </el-form-item>
              <el-form-item label="划线价格" prop="scribingPrice">
                <el-input v-model="saveProductSpecForm.scribingPrice" placeholder="请输入划线价格"></el-input>
              </el-form-item>
              <el-form-item label="成本价格" prop="costPrice">
                <el-input v-model="saveProductSpecForm.costPrice" placeholder="请输入成本价格"></el-input>
              </el-form-item>
            </el-form>
            <div class="demo-drawer__footer" style="text-align:center;">
              <el-button type="primary" @click="saveProductSpec()" :loading="saveSpecLoading">确 定</el-button>
              <el-button @click="innerDrawerClose()">取 消</el-button>
            </div>
          </el-drawer>
        </div>
      </el-drawer>
    </div>
  </div>
</template>
<script>
import { quillEditor } from "vue-quill-editor";
import quillConfig from "../../utils/quill-config.js";
export default {
  components: {
    quillEditor
  },
  name: "product",
  data() {
    // eslint-disable-next-line no-unused-vars
    let _self = this;
    let typeIdValidator = (rule, value, callback) => {
      // eslint-disable-next-line no-console
      if (value[0] === 0) {
        callback(new Error("请选择分类"));
      }
      callback();
    };
    return {
      transferTittle: ["规格列表", "已选择规格"],
      selectProductId: "",
      productSpecDrawer: false,
      dialogCentr: true,
      labelPosition: "left",
      quillOption: quillConfig,
      query: {
        pageNum: 1,
        pageSize: 10
      },
      tableData: [],
      pageTotal: 0,
      editVisible: false,
      saveForm: {},
      dialogMessage: "",
      addOrUpdate: 0,
      idx: -1,
      id: -1,
      productStatusList: [
        { id: 0, name: "未发布" },
        { id: 1, name: "已发布" },
        { id: 2, name: "已成交" },
        { id: 3, name: "已下架" }
      ],
      productTypeList: [
        { id: 1, name: "新手商品" },
        { id: 2, name: "正常商品" }
      ],
      recommendTypeData: [],
      categoryList: [],
      dialogImageUrl: "",
      imgDialogVisible: false,
      fileLists: [],
      detailFileLists: [],
      homeFileLists: [],
      ruleValidate: {
        productName: [
          { required: true, message: "请输入商品名称", trigger: "blur" }
        ],
        typeId: [
          { required: true, message: "请选择商品分类", trigger: "change" },
          { validator: typeIdValidator, trigger: "blur" }
        ],
        productTag: [
          { required: true, message: "请输入商品标签", trigger: "blur" }
        ],
        productPrice: [
          { required: true, message: "请输入价格", trigger: "blur" }
        ],
        totalNumber: [
          { required: true, message: "请输入任务线总数", trigger: "blur" }
        ],
        productNumber: [
          { required: true, message: "请输入商品数量", trigger: "blur" }
        ],
        totalReward: [
          { required: true, message: "请输入总佣金", trigger: "blur" }
        ],
        taskPrice: [
          { required: true, message: "请输入任务单价", trigger: "blur" }
        ],
        taskNumber: [
          { required: true, message: "请输入任务份数", trigger: "blur" }
        ],
        productStatus: [
          { required: true, message: "请选择商品状态", trigger: "change" }
        ],
        productType: [
          { required: true, message: "请选择产品类型", trigger: "change" }
        ],
        shopId: [{ required: true, message: "请选择商户", trigger: "change" }]
      },
      specRuleValidate: {
        productNumber: [
          { required: true, message: "请输入商品数量", trigger: "blur" }
        ],
        productPrice: [
          { required: true, message: "请输入商品价格", trigger: "blur" }
        ],
        costPrice: [
          { required: true, message: "请输入成本数量", trigger: "blur" }
        ],
        scribingPrice: [
          { required: true, message: "请输入划线价格", trigger: "blur" }
        ]
      },
      productSpecQuery: {},
      productSpecList: [],
      saveProductSpecForm: {},
      innerDrawerTittle: "",
      innerDrawer: false,
      productSpecAddOrUpdate: 0,
      productSpecValueList: [],
      selectProductSpecValueList: [],
      tableDataLoading: true,
      innerTableDataLoading: true,
      shopList: [],
      totalNumberDisabled: false,
      taskNumberDisabled: false,
      taskPriceDisabled: false,
      totalRewardDisabled: false,
      saveLoading: false,
      saveSpecLoading: false,
      props: {
        lazy: true,
        lazyLoad: this.cascaderlazyLoad
      },
      selectProps: {
        lazy: true,
        lazyLoad: this.cascaderlazyLoad
      },
      multipleSelection: [],
      selectRecommendType: "",
      selectActivity: "",
      activityData: [],
      shopDisabled: false,
      recommendTypeDisabled: false,
      activityDisabled: false,
      loginUser: {},
      editShopDisabled: false
    };
  },
  created() {
    this.handleUserShop();
    this.getTableData();
    this.getCategoryList();
    this.getShopListData();
    this.getRecommendTypeData();
    this.getActivityData();
  },
  methods: {
    handleUserShop() {
      this.loginUser = JSON.parse(window.localStorage.getItem("user"));
      if (this.loginUser.shopId !== undefined) {
        //处理权限问题
        this.query.shopId = parseInt(this.loginUser.shopId);
        this.shopDisabled = true;
        this.recommendTypeDisabled = true;
        this.activityDisabled = true;
      }
    },
    //推荐分类列表
    getRecommendTypeData() {
      var url = "/product/listRecommendType";
      this.postRequest(url, {}).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.recommendTypeData = resp.data.obj;
        }
      });
    },
    //活动列表
    getActivityData() {
      var url = "/activity/listActivity";
      this.postRequest(url, {
        pageNum: 1,
        pageSize: 100
      }).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.activityData = resp.data.obj.list;
        }
      });
    },
    //多选框
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    //级联选择起获取数据
    cascaderlazyLoad(node, resolve) {
      const { level } = node;
      const { data } = node;
      var typeId = "";
      if (level !== 0) {
        typeId = data.value;
      }
      var params = { value: typeId };
      this.postRequest("/category/listCategory", params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          let nodes = resp.data.obj;
          resolve(nodes);
        }
      });
    },

    //添加滑出
    handleProductSpecAdd() {
      this.saveSpecLoading = false;
      this.innerDrawerTittle = "添加商品规格";
      this.productSpecAddOrUpdate = 0;
      this.innerDrawer = true;
      this.getProductSpecValueTableData();
    },
    //编辑滑出
    handleProductSpecEdit(index, row) {
      this.saveSpecLoading = false;
      this.innerDrawerTittle = "编辑商品规格";
      this.productSpecAddOrUpdate = 1;
      this.saveProductSpecForm = JSON.parse(JSON.stringify(row));
      this.innerDrawer = true;
      this.getProductSpecValueTableData();
      var specList = row.productSpecValueId.split("_");
      for (let i = 0; i < specList.length; i++) {
        this.selectProductSpecValueList.push(parseInt(specList[i]));
      }
    },
    /* eslint-disable no-unused-vars */
    handleInnerDrawerClose(done) {
      this.$confirm("确定关闭吗？")
        .then(_ => {
          this.resetForm("saveProductSpecForm");
          this.saveProductSpecForm = {};
          this.selectProductSpecValueList = [];
          done();
        })
        .catch(_ => {});
    },
    handleDrawerClose(done) {
      this.productSpecList = [];
      this.tableDataLoading = true;
      this.getTableData();
      done();
    },
    innerDrawerClose() {
      this.innerDrawer = false;
      this.resetForm("saveProductSpecForm");
      this.saveProductSpecForm = {};
      this.selectProductSpecValueList = [];
    },
    //保存商品规格
    saveProductSpec() {
      this.saveSpecLoading = true;
      var url = "";
      if (this.productSpecAddOrUpdate == 0) {
        url = "/product/insertProductSpec";
        this.saveProductSpecForm.productId = this.selectProductId;
      } else if (this.productSpecAddOrUpdate == 1) {
        url = "/product/updateProductSpec";
      }
      this.saveProductSpecForm.productSpecValueId = JSON.stringify(
        this.selectProductSpecValueList
      );
      var params = this.saveProductSpecForm;
      this.postRequest(url, params).then(resp => {
        this.saveSpecLoading = false;
        if (resp && resp.status == 200) {
          if (resp.data.status == "200") {
            this.$message.success("保存成功");
            this.innerDrawer = false;
            this.resetForm("saveProductSpecForm");
            this.saveProductSpecForm = {};
            this.getProductSpecTableData(this.selectProductId);
            this.selectProductSpecValueList = [];
          }
        }
      });
    },
    //删除商品规格
    handleProductSpecDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/product/deleteProductSpec";
          var params = { id: row.id };
          this.postRequest(url, params).then(resp => {
            if (resp && resp.status == 200 && resp.data.status == "200") {
              this.$message.success("删除成功");
              this.getProductSpecTableData(this.selectProductId);
            } else {
              this.$message("删除失败");
            }
          });
        })
        .catch(() => {});
    },
    //查看商品规格列表
    handleMore(index, row) {
      this.productSpecDrawer = row.id;
      this.productSpecDrawer = true;
      this.selectProductId = row.id;
      this.innerTableDataLoading = true;
      this.getProductSpecTableData(row.id);
    },
    //关闭弹窗
    closeDialog() {
      this.resetForm("saveForm");
      this.saveForm = {};
      this.fileLists = [];
      this.detailFileLists = [];
      this.homeFileLists = [];
    },
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then(_ => {
          done();
        })
        .catch(_ => {});
    },
    resetForm(formName) {
      this.$refs[formName].clearValidate();
      this.$refs[formName].resetFields();
    },
    //获取商品列表
    getTableData() {
      var url = "/product/listProduct";
      if (
        this.query.typeId !== null &&
        this.query.typeId !== undefined &&
        this.query.typeId !== ""
      ) {
        if (this.query.typeId[1] !== 0) {
          var typeId = this.query.typeId[1];
          this.query.typeId = typeId;
        } else {
          this.query.typeId = null;
        }
      } else {
        this.query.typeId = null;
      }
      this.postRequest(url, this.query).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.tableData = resp.data.obj.list;
          this.pageTotal = resp.data.obj.total;
          this.tableDataLoading = false;
        }
      });
    },
    getShopListData() {
      var url = "/product/listShop";
      var params = { pageNum: 1, pageSize: 10000 };
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.shopList = resp.data.obj.list;
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
    //商品规格列表
    getProductSpecTableData(productId) {
      this.productSpecQuery.id = productId;
      var params = this.productSpecQuery;
      var url = "/product/listProductSpec";
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          this.productSpecList = resp.data.obj;
          this.innerTableDataLoading = false;
        }
      });
    },
    //商品规格值列表
    getProductSpecValueTableData() {
      var params = {};
      var url = "/product/listProductSpecValue";
      this.postRequest(url, params).then(resp => {
        if (resp && resp.status == 200 && resp.data.status == "200") {
          var data = [];
          data = resp.data.obj;
          this.productSpecValueList = [];
          for (let i = 0; i < data.length; i++) {
            this.productSpecValueList.push({
              key: parseInt(data[i].id),
              label: data[i].valueType + "_" + data[i].valueName
            });
          }
        }
      });
    },
    //搜索功能
    handleSearch() {
      this.tableDataLoading = true;
      this.query.pageNum = 1;
      this.getTableData();
    },
    //重置搜索条件
    resetQuery() {
      this.query.productName = null;
      this.query.productStatus = null;
      this.query.productType = null;
      this.query.typeId = null;
      if (this.loginUser.shopId === undefined) {
        this.query.shopId = null;
      }
      this.tableDataLoading = true;
      this.getTableData();
    },
    //编辑弹出
    handleEdit(index, row) {
      this.saveLoading = false;
      this.dialogMessage = "编辑商品";
      this.addOrUpdate = 1;
      this.totalNumberDisabled = true;
      this.taskNumberDisabled = true;
      this.taskPriceDisabled = true;
      this.totalRewardDisabled = true;
      this.fileLists = [
        {
          name: "",
          url: row.productImg
        }
      ];
      this.homeFileLists = [
        {
          name: "",
          url: row.homeProductImg
        }
      ];
      if (row.productImgList.length > 0) {
        for (var i = 0; i < row.productImgList.length; i++) {
          this.detailFileLists.push({
            name: "",
            url: row.productImgList[i].imgUrl
          });
        }
      }
      this.saveForm = JSON.parse(JSON.stringify(row));
      if (this.loginUser.shopId !== undefined) {
        this.editShopDisabled = true;
      }
      this.saveForm.shopId = parseInt(this.saveForm.shopId);
      if (this.loginUser.shopId !== undefined) {
        this.editShopDisabled = true;
      }
      this.saveForm.typeId = JSON.parse(this.saveForm.typeId);
      this.editVisible = true;
    },
    //添加弹出
    handleAdd() {
      if (this.loginUser.shopId !== undefined) {
        this.saveForm.shopId = parseInt(this.loginUser.shopId);
        this.editShopDisabled = true;
      }
      this.saveLoading = false;
      this.dialogMessage = "添加商品";
      this.addOrUpdate = 0;
      this.totalNumberDisabled = false;
      this.taskNumberDisabled = false;
      this.taskPriceDisabled = false;
      this.totalRewardDisabled = false;
      this.editVisible = true;
    },
    //多选添加到推荐
    handleAddToRecommend() {
      if (this.selectRecommendType === "") {
        this.$message({
          showClose: true,
          message: "请选择要添加到的推荐分类",
          type: "warning"
        });
      } else {
        if (this.multipleSelection.length <= 0) {
          this.$message({
            showClose: true,
            message: "请选择要添加推荐分类的商品",
            type: "warning"
          });
        } else {
          var params = {
            recommendTypeId: this.selectRecommendType,
            productList: this.multipleSelection
          };
          var url = "product/insertRecommendProductList";
          this.postRequest(url, params).then(resp => {
            if (resp && resp.status == 200 && resp.data.status == "200") {
              this.$message.success("添加成功");
              this.getTableData();
            } else {
              this.$message("添加失败");
            }
          });
        }
      }
    },
    //添加到活动
    handleAddToActivity() {
      if (this.selectActivity === "") {
        this.$message({
          showClose: true,
          message: "请选择要添加到的活动",
          type: "warning"
        });
      } else {
        if (this.multipleSelection.length <= 0) {
          this.$message({
            showClose: true,
            message: "请选择要添加活动的商品",
            type: "warning"
          });
        } else {
          var params = {
            actId: this.selectActivity,
            productList: this.multipleSelection
          };
          var url = "/activity/insertActivityProductList";
          this.postRequest(url, params).then(resp => {
            if (resp && resp.status == 200 && resp.data.status == "200") {
              this.$message.success("添加成功");
              this.getTableData();
            } else {
              this.$message("添加失败");
            }
          });
        }
      }
    },
    //删除商品
    handleDelete(index, row) {
      this.$confirm("确定要删除吗？", "提示", {
        type: "warning"
      })
        .then(() => {
          var url = "/product/deleteProduct";
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
    //分页
    handlePageChange(val) {
      this.tableDataLoading = true;
      this.$set(this.query, "pageNum", val);
      this.getTableData();
    },
    //商品状态处理
    // eslint-disable-next-line no-unused-vars
    statusFormat(row, column) {
      if (row.productStatus == 0) {
        return "未发布";
      } else if (row.productStatus == 1) {
        return "已发布";
      } else if (row.productStatus == 2) {
        return "已成交";
      } else if (row.productStatus == 3) {
        return "已下架";
      }
    },
    //产品类型处理
    // eslint-disable-next-line no-unused-vars
    typeFormat(row, column) {
      if (row.productType == 1) {
        return "新手商品";
      } else if (row.productType == 2) {
        return "正常商品";
      }
    },
    //格式化列表日期
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
    tableHeaderColor(row, rowIndex) {
      return "background-color:#EBEEF5;";
    },
    //保存商品
    saveProduct() {
      this.$refs.saveForm.validate(valid => {
        if (valid) {
          this.saveLoading = true;
          var url = "";
          if (this.addOrUpdate == 0) {
            url = "/product/insertProduct";
          } else if (this.addOrUpdate == 1) {
            url = "/product/updateProduct";
          }
          if (this.fileLists.length <= 0) {
            this.$message("请上传商品主图");
            this.saveLoading = false;
          }
          this.saveForm.productImg = this.fileLists[0].url;
          if (this.homeFileLists.length >= 1) {
            this.saveForm.homeProductImg = this.homeFileLists[0].url;
          }
          var productImgList = [];
          for (var i = 0; i < this.detailFileLists.length; i++) {
            productImgList.push({ imgUrl: this.detailFileLists[i].url });
          }
          this.saveForm.productImgList = productImgList;
          this.saveForm.typeId = this.saveForm.typeId[1];
          var params = this.saveForm;
          this.postRequest(url, params).then(resp => {
            this.saveLoading = false;
            if (resp && resp.status == 200 && resp.data.status == "200") {
              this.$message.success("保存成功");
              this.editVisible = false;
              this.getTableData();
            } else {
              this.$message("保存失败");
              this.saveLoading = false;
            }
          });
        } else {
          return false;
        }
      });
    },
    //删除图片
    // eslint-disable-next-line no-unused-vars
    handleRemove(file, fileList) {
      this.fileLists = [];
    },
    handleHomeRemove(file, fileList) {
      this.homeFileLists = [];
    },
    handleDetailRemove(file) {
      for (var i = 0; i < this.detailFileLists.length; i++) {
        if (this.detailFileLists[i].url == file.url) {
          this.detailFileLists.splice(i, 1);
          break;
        }
      }
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
          url: file.response.url
        }
      ];
    },
    homeSuccess(response, file, fileList) {
      this.homeFileLists = [
        {
          name: file.name,
          url: file.response.url
        }
      ];
    },
    detailSuccess(response, file, fileList) {
      this.detailFileLists.push({
        name: file.name,
        url: file.response.url
      });
    },
    // eslint-disable-next-line no-unused-vars
    handleExceed(files, fileList) {
      this.$message.warning(`请最多上传一个图片。`);
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
.handle-box {
  margin-bottom: 10px;
  text-align: left;
}
.name-input {
  width: 180px;
  margin-right: 10px;
  margin-top: 10px;
}
.type-select {
  margin-right: 10px;
  width: 180px;
}
.add-button {
  padding-right: 0px;
}
.inner-drawer-tittle {
  font-size: 18px;
  color: #303133;
  line-height: 50px;
}
.query-button {
  margin-top: 10px;
}
</style>