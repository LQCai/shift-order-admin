<template>
  <basic-container>
    <avue-crud :option="option"
               :table-loading="loading"
               :data="data"
               ref="crud"
               v-model="form"
               :permission="permissionList"
               :search.sync="search"
               @row-del="rowDel"
               @row-update="rowUpdate"
               @row-save="rowSave"
               :before-open="beforeOpen"
               :page="page"
               @search-change="searchChange"
               @search-reset="searchReset"
               @selection-change="selectionChange"
               @current-change="currentChange"
               @size-change="sizeChange"
               @refresh-change="refreshChange"
               @on-load="onLoad">
      <template slot="menuLeft">
        <el-button type="danger"
                   size="small"
                   icon="el-icon-delete"
                   plain
                   v-if="permission.user_delete"
                   @click="handleDelete">删 除
        </el-button>
        <el-button type="info"
                   size="small"
                   plain
                   v-if="userInfo.authority.includes('admin')"
                   icon="el-icon-user"
                   @click="handleGrant">角色配置
        </el-button>
        <el-button type="primary"
                   size="small"
                   plain
                   v-if="permission.user_reset"
                   icon="el-icon-refresh"
                   @click="handleReset">密码重置
        </el-button>
        <el-button type="success"
                   size="small"
                   plain
                   v-if="userInfo.authority.includes('admin')"
                   icon="el-icon-upload2"
                   @click="handleImport">导入
        </el-button>
        <el-button type="warning"
                   size="small"
                   plain
                   v-if="userInfo.authority.includes('admin')"
                   icon="el-icon-download"
                   @click="handleExport">导出
        </el-button>
        <el-button type="info"
                   size="small"
                   plain
                   v-if="userInfo.authority.includes('admin')"
                   icon="el-icon-user"
                   @click="handleReview">审核
        </el-button>
      </template>
      <template slot-scope="{row}"
                slot="roleId">
        <el-tag>{{row.roleName}}</el-tag>
      </template>
      <template slot-scope="{row}"
                slot="statusName">
        <el-tag v-if="row.status === 0">{{row.statusName}}</el-tag>
        <el-tag type="success" v-if="row.status === 1">{{row.statusName}}</el-tag>
        <el-tag type="danger" v-if="row.status === 2">{{row.statusName}}</el-tag>
      </template>
    </avue-crud>
    <el-dialog title="用户角色配置"
               append-to-body
               :visible.sync="roleBox"
               width="345px">

      <el-tree :data="roleGrantList"
               show-checkbox
               default-expand-all
               node-key="id"
               ref="treeRole"
               :default-checked-keys="roleTreeObj"
               :props="props">
      </el-tree>

      <span slot="footer" class="dialog-footer">
            <el-button @click="roleBox = false">取 消</el-button>
            <el-button type="primary"
                       @click="submitRole">确 定</el-button>
          </span>
    </el-dialog>
    <el-dialog title="用户数据导入"
               append-to-body
               :visible.sync="excelBox"
               width="555px">
      <avue-form :option="excelOption" v-model="excelForm" :upload-after="uploadAfter">
        <template slot="excelTemplate">
          <el-button type="primary" @click="handleTemplate()">
            点击下载<i class="el-icon-download el-icon--right"></i>
          </el-button>
        </template>
      </avue-form>
    </el-dialog>
    <el-dialog title="用户审批"
               append-to-body
               :visible.sync="reviewVisible"
               width="555px">

      <div v-for="item in selectionList">
        <a>姓名: {{ item.name }}; </a>
        <a>工号: {{ item.code }}; </a>
        <a>电话: {{ item.phone }} </a>
      </div>

      <span slot="footer" class="dialog-footer">
            <el-button @click="reviewVisible = false">取 消</el-button>
            <el-button type="danger" @click="submitReview(2)">拒 绝</el-button>
            <el-button type="primary"
                       @click="submitReview(1)">通 过</el-button>
          </span>
    </el-dialog>
  </basic-container>
</template>

<script>
import {
  getList,
  getUser,
  remove,
  update,
  add,
  grant,
  resetPassword, review
} from "@/api/system/user";
  import {getRoleTree} from "@/api/system/role";
  import {mapGetters} from "vuex";
  import website from '@/config/website';
  import {getToken} from '@/util/auth';

  export default {
    data() {
      const validatePass = (rule, value, callback) => {
        if (value === '') {
          callback(new Error('请输入密码'));
        } else {
          callback();
        }
      };
      const validatePass2 = (rule, value, callback) => {
        if (value === '') {
          callback(new Error('请再次输入密码'));
        } else if (value !== this.form.password) {
          callback(new Error('两次输入密码不一致!'));
        } else {
          callback();
        }
      };
      return {
        reviewVisible: false,
        form: {},
        search:{},
        roleBox: false,
        excelBox: false,
        loading: true,
        selectionList: [],
        query: {},
        page: {
          pageSize: 10,
          currentPage: 1,
          total: 0
        },
        props: {
          label: "title",
          value: "key"
        },
        roleGrantList: [],
        roleTreeObj: [],
        option: {
          height: 'auto',
          calcHeight: 210,
          searchShow: true,
          searchMenuSpan: 6,
          tip: false,
          border: true,
          index: true,
          selection: true,
          viewBtn: true,
          column: [
            {
              label: "登录账号",
              prop: "account",
              search: true,
              rules: [{
                required: true,
                message: "请输入登录账号",
                trigger: "blur"
              }],
              span: website.tenantMode ? 12 : 24,
            },
            {
              label: "所属租户",
              prop: "tenantId",
              type: "tree",
              dicUrl: "/api/blade-system/tenant/select",
              props: {
                label: "tenantName",
                value: "tenantId"
              },
              hide: !website.tenantMode,
              addDisplay: website.tenantMode,
              editDisplay: website.tenantMode,
              viewDisplay: website.tenantMode,
              search: false,
              rules: [{
                required: true,
                message: "请输入所属租户",
                trigger: "click"
              }]
            },
            {
              label: '密码',
              prop: 'password',
              hide: true,
              editDisplay: false,
              viewDisplay: false,
              rules: [{required: true, validator: validatePass, trigger: 'blur'}]
            },
            {
              label: '确认密码',
              prop: 'password2',
              hide: true,
              editDisplay: false,
              viewDisplay: false,
              rules: [{required: true, validator: validatePass2, trigger: 'blur'}]
            },
            {
              label: "用户昵称",
              prop: "name",
              rules: [{
                required: true,
                message: "请输入用户昵称",
                trigger: "blur"
              }]
            },
            {
              label: "用户姓名",
              prop: "realName",
              search: true,
              rules: [{
                required: true,
                message: "请输入用户姓名",
                trigger: "blur"
              }]
            },
            {
              label: "状态",
              prop: "statusName",
              addDisplay: false,
              editDisplay: false,
              slot: true
            },
            {
              label: "所属角色",
              prop: "roleId",
              multiple: true,
              type: "tree",
              dicData: [],
              props: {
                label: "title"
              },
              slot: true,
              checkStrictly: true,
              rules: [{
                required: true,
                message: "请选择所属角色",
                trigger: "click"
              }]
            },
            {
              label: "用户编号",
              prop: "code",
              hide: true,
            },
            {
              label: "手机号码",
              prop: "phone",
              overHidden: true
            },
            {
              label: "电子邮箱",
              prop: "email",
              hide: true,
              overHidden: true
            },
            {
              label: "用户性别",
              prop: "sex",
              type: "select",
              dicData: [
                {
                  label: "男",
                  value: 1
                },
                {
                  label: "女",
                  value: 2
                },
                {
                  label: "未知",
                  value: 3
                }
              ],
              hide: true
            },
            {
              label: "用户生日",
              type: "date",
              prop: "birthday",
              format: "yyyy-MM-dd hh:mm:ss",
              valueFormat: "yyyy-MM-dd hh:mm:ss",
              hide: true
            }
          ]
        },
        data: [],
        excelForm: {},
        excelOption: {
          submitBtn: false,
          emptyBtn: false,
          column: [
            {
              label: '模板上传',
              prop: 'excelFile',
              type: 'upload',
              drag: true,
              loadText: '模板上传中，请稍等',
              span: 24,
              propsHttp: {
                res: 'data'
              },
              tip: '请上传 .xls,.xlsx 标准格式文件',
              action: "/api/blade-user/import-user"
            },
            {
              label: '模板下载',
              prop: 'excelTemplate',
              formslot: true,
              span: 24,
            }
          ]
        }
      };
    },
    watch: {
      'form.tenantId'() {
        if (this.form.tenantId !== '') {
          getRoleTree(this.form.tenantId).then(res => {
            const column = this.findObject(this.option.column, "roleId");
            column.dicData = res.data.data;
          });
        }
      },
    },
    computed: {
      ...mapGetters(["userInfo", "permission"]),
      permissionList() {
        return {
          addBtn: this.vaildData(this.permission.user_add, false),
          viewBtn: this.vaildData(this.permission.user_view, false),
          delBtn: this.vaildData(this.permission.user_delete, false),
          editBtn: this.vaildData(this.permission.user_edit, false)
        };
      },
      ids() {
        let ids = [];
        this.selectionList.forEach(ele => {
          ids.push(ele.id);
        });
        return ids.join(",");
      },
    },
    methods: {
      submitRole() {
        const roleList = this.$refs.treeRole.getCheckedKeys().join(",");
        grant(this.ids, roleList).then(() => {
          this.roleBox = false;
          this.$message({
            type: "success",
            message: "操作成功!"
          });
          this.onLoad(this.page);
        });
      },
      rowSave(row, done, loading) {
        row.roleId = row.roleId.join(",");
        add(row).then(() => {
          done();
          this.onLoad(this.page);
          this.$message({
            type: "success",
            message: "操作成功!"
          });
        }, error => {
          window.console.log(error);
          loading();
        });
      },
      rowUpdate(row, index, done, loading) {
        row.roleId = row.roleId.join(",");
        update(row).then(() => {
          done();
          this.onLoad(this.page);
          this.$message({
            type: "success",
            message: "操作成功!"
          });
        }, error => {
          window.console.log(error);
          loading();
        });
      },
      rowDel(row) {
        this.$confirm("确定将选择数据删除?", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            return remove(row.id);
          })
          .then(() => {
            this.onLoad(this.page);
            this.$message({
              type: "success",
              message: "操作成功!"
            });
          });
      },
      searchReset() {
        this.query = {};
        this.onLoad(this.page);
      },
      searchChange(params, done) {
        this.query = params;
        this.page.currentPage = 1;
        this.onLoad(this.page, params);
        done();
      },
      selectionChange(list) {
        this.selectionList = list;
      },
      handleDelete() {
        if (this.selectionList.length === 0) {
          this.$message.warning("请选择至少一条数据");
          return;
        }
        this.$confirm("确定将选择数据删除?", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            return remove(this.ids);
          })
          .then(() => {
            this.onLoad(this.page);
            this.$message({
              type: "success",
              message: "操作成功!"
            });
            this.$refs.crud.toggleSelection();
          });
      },
      handleReset() {
        if (this.selectionList.length === 0) {
          this.$message.warning("请选择至少一条数据");
          return;
        }
        this.$confirm("确定将选择账号密码重置为123456?", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            return resetPassword(this.ids);
          })
          .then(() => {
            this.$message({
              type: "success",
              message: "操作成功!"
            });
            this.$refs.crud.toggleSelection();
          });
      },
      handleGrant() {
        if (this.selectionList.length === 0) {
          this.$message.warning("请选择至少一条数据");
          return;
        }
        this.roleTreeObj = [];
        if (this.selectionList.length === 1) {
          this.roleTreeObj = this.selectionList[0].roleId.split(",");
        }
        getRoleTree().then(res => {
          this.roleGrantList = res.data.data;
          this.roleBox = true;
        });
      },
      handleImport() {
        this.excelBox = true;
      },
      uploadAfter(res, done, loading, column) {
        window.console.log(column);
        done();
        this.excelBox = false;
        this.refreshChange();
      },
      handleExport() {
        this.$confirm("是否导出用户数据?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }).then(() => {
          window.open(`/api/blade-user/export-user?blade-auth=${getToken()}&account=${this.search.account}&realName=${this.search.realName}`);
        });
      },
      handleTemplate() {
        window.open(`/api/blade-user/export-template?blade-auth=${getToken()}`);
      },
      beforeOpen(done, type) {
        if (["edit", "view"].includes(type)) {
          getUser(this.form.id).then(res => {
            this.form = res.data.data;
            if(this.form.hasOwnProperty("roleId")){
              this.form.roleId = this.form.roleId.split(",");
            }
          });
        }
        done();
      },
      currentChange(currentPage) {
        this.page.currentPage = currentPage;
      },
      sizeChange(pageSize) {
        this.page.pageSize = pageSize;
      },
      refreshChange() {
        this.onLoad(this.page, this.query);
      },
      handleReview() {
        if (this.selectionList.length === 0) {
          this.$message.warning("请选择至少一条数据");
          return;
        }

        const statusList = this.selectionList.map(item => item.status)
        if (statusList.indexOf(1) !== -1 || statusList.indexOf(2) !== -1) {
          this.$message.warning("请勿勾选非申请状态的数据");
          return;
        }

        this.reviewVisible = true
      },
      submitReview(status) {
        review(this.ids, status).then(res => {
          this.onLoad(this.page);
          this.$message({
            type: "success",
            message: "操作成功!"
          });
        }).catch(e => {
          // ...
        });
        this.reviewVisible = false
      },
      onLoad(page, params = {}) {
        this.loading = true;
        getList(page.currentPage, page.pageSize, Object.assign(params, this.query)).then(res => {
          const data = res.data.data;
          this.page.total = data.total;
          this.data = data.records;
          this.loading = false;
        });
        getRoleTree(this.form.tenantId).then(res => {
          const column = this.findObject(this.option.column, "roleId");
          column.dicData = res.data.data;
        });
      }
    }
  };
</script>

<style>
</style>
