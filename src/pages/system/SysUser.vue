<template>
  <div class="sysuser-container" v-loading="loading">
    <div class="header-container">
      <el-input
        class="search-content"
        placeholder="默认展示部分用户，可以通过用户名搜索更多用户..."
        prefix-icon="el-icon-search"
        size="small"
        v-model="keyWords"
      ></el-input>
      <el-button size="small" type="primary" icon="el-icon-search" @click="searchClick">搜索</el-button>
      <el-button
        style="margin-left:10px;"
        size="small"
        type="primary"
        icon="el-icon-plus"
        @click="addUserDialogShow"
      >添加用户</el-button>
    </div>
    <div class="body-container">
      <div class="card-container" v-for="(user,index) of users" :key="user.id">
        <el-card class="user-card" v-loading="cardLoadings[index]">
          <div slot="header" class="clearfix">
            <span>{{user.name}}</span>
            <el-button
              class="button"
              type="text"
              icon="el-icon-delete"
              @click="deleteUser(user.id)"
            ></el-button>
            <el-button
              class="button"
              type="text"
              icon="el-icon-edit"
              @click="showUpdateUserView(user)"
              style="margin-right: 10px"
            ></el-button>
          </div>
          <div>
            <div class="img-container">
              <img class="user-img" :src="user.userface" :alt="user.name" />
            </div>
            <div class="info-container">
              <span class="user-info">用户名：{{user.name}}</span>
              <span class="user-info">手机号码：{{user.phone}}</span>
              <span class="user-info">电话号码：{{user.telephone}}</span>
              <span class="user-info">地址：{{user.address}}</span>
              <div class="user-info user-state">
                用户状态：
                <el-switch
                  class="switch"
                  v-model="user.enabled"
                  active-color="#13ce66"
                  inactive-color="#aaaaaa"
                  active-text="启用"
                  inactive-text="禁用"
                  :key="user.id"
                  @change="switchChange(user.enabled,user.id,index)"
                ></el-switch>
              </div>
              <div class="user-info">
                用户角色：
                <el-tag
                  class="role-tag"
                  v-for="role of user.roles"
                  :key="role.id"
                  type="success"
                  size="mini"
                  :disable-transitions="false"
                >{{role.nameZh}}</el-tag>
                <el-popover
                  v-loading="epLoadings[index]"
                  placement="right"
                  title="角色列表"
                  width="200"
                  trigger="click"
                  @hide="updateUserRoles(user.id,index)"
                >
                  <el-select v-model="selectedRoles" multiple placeholder="请选择角色">
                    <el-option
                      v-for="ar in allRoles"
                      :key="ar.id"
                      :label="ar.nameZh"
                      :value="ar.id"
                    ></el-option>
                  </el-select>
                  <!-- slot="reference"：触发 popover -->
                  <el-button
                    class="more-button"
                    type="text"
                    icon="el-icon-more"
                    slot="reference"
                    @click="loadSelectedRoles(user.roles,index)"
                    :disabled="moreBtnState"
                  ></el-button>
                </el-popover>
              </div>
              <span class="user-info">商户名称：{{user.shopName}}</span>
              <span class="user-info">描述：{{user.remark}}</span>
            </div>
          </div>
        </el-card>
      </div>
    </div>
    <div v-if="total>pageSize" class="foot-container">
      <el-pagination
        background
        :page-size="pageSize"
        :current-page="pageNum"
        @current-change="handleCurrentChange"
        layout="total, prev, pager, next, jumper"
        :page-count="7"
        :total="total"
      ></el-pagination>
    </div>
    <div class="dialog-container">
      <el-dialog
        :title="dialogTitle"
        :visible.sync="dialogVisible"
        width="50%"
        @close="closeDialog"
      >
        <el-form
          :model="userForm"
          ref="userForm"
          :rules="rules"
          status-icon
          label-width="100px"
          label-position="right"
          v-loading="formLoading"
        >
          <el-row>
            <el-col :span="12">
              <div>
                <el-form-item prop="name" label="姓名：">
                  <el-input
                    prefix-icon="el-icon-edit"
                    v-model="userForm.name"
                    size="mini"
                    placeholder="请输入姓名..."
                  ></el-input>
                </el-form-item>
              </div>
            </el-col>
            <el-col :span="11" :offset="1">
              <div>
                <el-form-item prop="phone" label="手机号码：">
                  <el-input
                    prefix-icon="el-icon-mobile-phone"
                    v-model="userForm.phone"
                    size="mini"
                    placeholder="请输入手机号码..."
                  ></el-input>
                </el-form-item>
              </div>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item prop="isShopAccount" label="商户账号：">
                <el-radio-group v-model="userForm.isShopAccount" @change="isShopAccountChange">
                  <el-radio label="1" border size="mini">是</el-radio>
                  <el-radio label="2" border size="mini">否</el-radio>
                </el-radio-group>
              </el-form-item>
            </el-col>
            <el-col :span="11" :offset="1">
              <el-form-item prop="shopId" label="选择商户：">
                <el-select
                  v-model="userForm.shopId"
                  :disabled="shopDisabled"
                  size="mini"
                  placeholder="请选择商户"
                >
                  <el-option
                    v-for="item in shopList"
                    :key="item.id"
                    :label="item.shopName"
                    :value="item.id"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <div>
                <el-form-item prop="address" label="地址：">
                  <el-input
                    prefix-icon="el-icon-edit"
                    v-model="userForm.address"
                    size="mini"
                    placeholder="请输入地址..."
                  ></el-input>
                </el-form-item>
              </div>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <div>
                <el-form-item prop="username" label="用户名：">
                  <el-input
                    prefix-icon="el-icon-edit"
                    v-model="userForm.username"
                    size="mini"
                    placeholder="请输入用户名..."
                  ></el-input>
                </el-form-item>
              </div>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="11">
              <div>
                <el-form-item prop="password" label="密码：">
                  <el-input
                    prefix-icon="el-icon-edit"
                    v-model="userForm.password"
                    show-password
                    size="mini"
                    placeholder="请输入密码..."
                  ></el-input>
                </el-form-item>
              </div>
            </el-col>
            <el-col :span="11" :offset="2">
              <div>
                <!-- 自定义校验规则，v-model 和 prop 属性必须按照这样的模式来写，有变化则会报错-->
                <el-form-item prop="checkPass" label="确认密码：">
                  <el-input
                    prefix-icon="el-icon-edit"
                    v-model="userForm.checkPass"
                    show-password
                    size="mini"
                    placeholder="请再次输入密码..."
                  ></el-input>
                </el-form-item>
              </div>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <div>
                <el-form-item label="描述：">
                  <el-input
                    prefix-icon="el-icon-edit"
                    v-model="userForm.remark"
                    size="mini"
                    placeholder="请输入描述..."
                  ></el-input>
                </el-form-item>
              </div>
            </el-col>
          </el-row>
        </el-form>
        <span slot="footer">
          <el-button size="samll" @click="cancel('userForm')">取消</el-button>
          <el-button size="samll" type="primary" @click="editUser('userForm')">确定</el-button>
        </span>
      </el-dialog>
    </div>
  </div>
</template>
<script>
export default {
  name: "SysUser",
  data() {
    var validatePass = (rule, value, callback) => {
      if (value === "") {
        callback(new Error("请再次输入密码"));
      } else if (value !== this.userForm.password) {
        callback(new Error("请两次输入密码不一致！"));
      } else {
        callback();
      }
    };
    var validateShopId = (rule, value, callback) => {
      if (this.userForm.isShopAccount == "1") {
        if (value === "" || value == null) {
          callback(new Error("请选择商户"));
        } else {
          callback();
        }
      } else {
        callback();
      }
    };
    return {
      loading: false,
      keyWords: "",
      pageNum: 1,
      pageSize: 6,
      total: 0,
      users: [],
      allRoles: [],
      // 用户卡片加载状态数组
      cardLoadings: [],
      // 角色pop框加载状态数组
      epLoadings: [],
      //本次选中的角色数组
      selectedRoles: [],
      //没选择角色之前，该用户所拥有的的角色
      oldRoles: [],
      // 是否禁用“更多”按钮
      moreBtnState: false,
      currentPage: 1,
      dialogVisible: false,
      dialogTitle: "",
      rules: [],
      formLoading: false,
      userForm: {
        name: "",
        phone: "",
        telephone: "",
        address: "",
        username: "",
        password: "",
        userface: "",
        remark: "",
        checkPass: "",
        shopId: "",
        isShopAccount: ""
      },
      shopList: [],
      shopDisabled: false,
      /* eslint-disable */
      rules: {
        name: [{ required: true, message: "请输入姓名", trigger: "blur" }],
        phone: [
          { required: true, message: "请输入手机号码", trigger: "blur" },
          { min: 11, max: 11, message: "必须为11位手机号码", trigger: "blur" }
        ],
        telephone: [
          { required: false, message: "请输入电话号码", trigger: "blur" }
        ],
        address: [{ required: true, message: "请输入地址", trigger: "blur" }],
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" }
        ],
        password: [{ required: false, message: "请输入密码", trigger: "blur" }],
        checkPass: [{ validator: validatePass, trigger: "blur" }],
        isShopAccount: [
          { required: true, message: "请选择是否商户账号", trigger: "change" }
        ],
        shopId: [{ validator: validateShopId, trigger: "change" }]
      }
    };
  },
  created() {
    this.getShopListData();
  },
  methods: {
    //关闭弹窗
    closeDialog() {
      this.resetForm("userForm");
      this.userForm = {};
    },
    resetForm(formName) {
      this.$refs[formName].clearValidate();
      this.$refs[formName].resetFields();
    },
    /**
     * @param keyWords 搜索词
     * @param pageNum 请求页
     */
    search(keyWords, pageNum) {
      this.pageNum = pageNum;
      this.loading = true;
      var _this = this;
      var searchWords;
      if (this.keyWords === "") {
        searchWords = "all";
      } else {
        searchWords = this.keyWords;
      }
      this.getRequestWithParams("/system/user/" + searchWords, {
        pageNum: pageNum,
        pageSize: this.pageSize
      }).then(resp => {
        _this.loading = false;
        if (resp && resp.status == 200) {
          _this.users = resp.data.obj.users;
          _this.total = resp.data.obj.total;
          var length = resp.data.obj.users.length;
          // 下面其实就是构造一个length长度的数组：[false, false, false...]
          // eslint-disable-next-line no-unused-vars
          _this.cardLoadings = Array.apply(null, Array(length)).map(function(
            item,
            i
          ) {
            return false;
          });
          // eslint-disable-next-line no-unused-vars
          _this.epLoadings = Array.apply(null, Array(length)).map(function(
            item,
            i
          ) {
            return false;
          });
        }
      });
    },
    isShopAccountChange(value) {
      if (value == "1") {
        this.shopDisabled = false;
      } else if (value === "2") {
        this.userForm.shopId = null;
        this.shopDisabled = true;
      }
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
    searchClick() {
      this.search(this.keyWords, 1);
    },
    addUserDialogShow() {
      this.dialogVisible = true;
      this.dialogTitle = "新增用户";
    },
    initUserCards() {
      this.search(this.keyWords, 1);
    },
    loadAllRoles() {
      var _this = this;
      this.getRequest("/system/basic/roles").then(resp => {
        if (resp && resp.status == 200) {
          _this.allRoles = resp.data.obj;
        }
      });
    },
    showUpdateUserView(user) {
      this.userForm = user;
      this.dialogVisible = true;
      this.userForm.shopId = "";
      this.dialogTitle = "更新用户";
    },
    deleteUser(userId) {
      var _this = this;
      this.loading = true;
      this.deleteRequest("/system/user/" + userId).then(resp => {
        _this.loading = false;
        // 若删除成功，重新请求数据
        if (resp && resp.status == 200 && resp.data.status == 200) {
          _this.initUserCards();
        }
      });
    },
    switchChange(enabled, userId, index) {
      // 该用户卡片对应的进度条状态设置为true
      this.cardLoadings.splice(index, 1, true);
      var _this = this;
      this.putRequest("/system/user/", {
        id: userId,
        enabled: enabled
      }).then(resp => {
        _this.cardLoadings.splice(index, 1, false);
        if (resp && resp.status == 200) {
          // 若更新失败，则请求更新前的数据（因为我们已经点击了switch，界面已经变化，只有把数据更换成更新前的数据，界面才会正常）
          if (resp.data.status == 500) {
            _this.refreshUser(userId, index);
          }
        } else {
          _this.refreshUser(userId, index);
        }
      });
    },
    refreshUser(userId, index) {
      this.cardLoadings.splice(index, 1, true);
      var _this = this;
      this.getRequest("/system/user/id/" + userId).then(resp => {
        this.cardLoadings.splice(index, 1, false);
        if (resp && resp.status == 200) {
          var user = resp.data.obj;
          // 将该位置的用户数据更新为请求来的数据
          _this.users.splice(index, 1, user);
        }
      });
    },
    updateUserRoles(userId, index) {
      //更多按钮可用
      this.moreBtnState = false;
      // //判断角色是否有变化，有变化则更新到后台；无变化，就不用
      // //依靠记录的 “选择前角色集合” 和 “选择后角色集合” 来判断有无变化
      // if (this.selectedRoles.length == this.oldRoles.length) {
      //     for (let i = 0; i < this.selectedRoles.length; i++) {
      //         for (let j = 0; j < this.oldRoles.length; j++) {
      //             if (this.selectedRoles[i] == this.oldRoles[j]) {
      //                 this.oldRoles.splice(j,1)
      //                 break
      //             }
      //         }
      //         // 若完整遍历 this.oldRoles 一次之后，长度依然相等，说明this.selectedRoles第i个元素
      //         // 和this.oldRoles集合中的元素没一个相等的,说明有变化，直接中断循环
      //         if (this.selectedRoles.length == this.oldRoles.length) {
      //             break
      //         }
      //     }
      // }
      // //若完全一致，则说明无变化，直接返回即可
      // if (this.oldRoles.length == 0) {
      //     return
      // }
      // 若执行到这一步，说明有变化，执行更新

      this.epLoadings.splice(index, 1, true);
      var _this = this;
      this.putRequest("/system/user/roles", {
        userId: userId,
        roleIds: this.selectedRoles
      }).then(resp => {
        _this.epLoadings.splice(index, 1, false);
        // 若更新成功，刷新数据，将新的角色展示出来（否则显示的还是旧的角色）
        if (resp && resp.status == 200 && resp.data.status == 200) {
          _this.refreshUser(userId, index);
        }
      });
    },
    // eslint-disable-next-line no-unused-vars
    loadSelectedRoles(roles, index) {
      this.moreBtnState = true;
      this.selectedRoles = [];
      this.oldRoles = [];
      roles.forEach(role => {
        // 因为 <el-select>的,<el-option>的绑定值是id：:value="ar.id"
        this.selectedRoles.push(role.id);
        this.oldRoles.push(role.id);
      });
    },
    handleCurrentChange(currentPage) {
      // 因为 currentPage 已经与 this.pageNum绑定了，所以我们不用再给其赋值
      this.search(this.keyWords, currentPage);
    },
    // eslint-disable-next-line no-unused-vars
    cancel(formName) {
      this.dialogVisible = false;
    },
    editUser(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.formLoading = true;
          var _this = this;
          if (this.userForm.id) {
            this.putRequest("/system/user/", this.userForm).then(resp => {
              _this.handleResponse(_this, resp, formName);
            });
          } else {
            this.postRequest("/system/user/", this.userForm).then(resp => {
              _this.handleResponse(_this, resp, formName);
            });
          }
        } else {
          return false;
        }
      });
    },
    handleResponse(context, resp, formName) {
      context.formLoading = false;
      if (resp && resp.status == 200 && resp.data.status == 200) {
        context.initUserCards();
        context.dialogVisible = false;

        // 成功以后，对表单重置，清空字段和校验结果(防止dialog还未加载完毕，就调用，报错！)
        if (context.$refs[formName] !== undefined) {
          context.$refs[formName].resetFields();
        }
        context.emptyUser();
      }
    },
    emptyUser() {
      this.userForm = {
        name: "",
        phone: "",
        telephone: "",
        address: "",
        username: "",
        password: "",
        userface: "",
        remark: "",
        checkPass: ""
      };
    }
  },
  mounted() {
    this.initUserCards();
    this.loadAllRoles();
  }
};
</script>
<style lang="stylus" scoped>
.sysuser-container {
  margin-top: 10px;

  .header-container {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 20px;

    .search-content {
      width: 400px;
      margin-right: 10px;
    }
  }

  .body-container {
    display: flex;
    justify-content: space-around;
    // 当前行不足以容纳所有元素时，进行换行
    flex-wrap: wrap;
    text-align: left;
    width: 100%;

    .card-container {
      flex: 0 0 33%;
      display: flex;
      justify-content: center;

      .user-card {
        width: 350px;
        margin-bottom: 20px;

        .button {
          float: right;
          padding: 3px 0;
          margin: 0;
          width: 15px;
          height: 15px;
          color: #f6061b;
        }

        .img-container {
          display: flex;
          justify-content: center;
          width: 100%;

          .user-img {
            width: 70px;
            height: 70px;
            border-radius: 70px;
          }
        }

        .info-container {
          margin-top: 20px;
          display: flex;
          flex-direction: column;

          .user-info {
            font-size: 14px;
            color: #09c0f6;

            .role-tag {
              margin-right: 5px;
            }

            .more-button {
              color: #09c0f6;
              padding-top: 0px;
            }
          }

          .user-state {
            display: flex;
            align-items: center;
            margin-bottom: 3px;

            .switch {
              display: inline;
              margin-left: 5px;
            }
          }
        }
      }
    }
  }

  .foot-container {
    display: flex;
    justify-content: center;
    margin-top: 10px;
  }

  .dialog-container {
    text-align: left;
  }
}
</style>
