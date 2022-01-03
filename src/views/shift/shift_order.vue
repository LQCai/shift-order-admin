<template>
  <basic-container>
    <avue-crud :option="option"
               :table-loading="loading"
               :data="data"
               :page="page"
               :permission="permissionList"
               :before-open="beforeOpen"
               v-model="form"
               ref="crud"
               @row-update="rowUpdate"
               @row-save="rowSave"
               @row-del="rowDel"
               :search.sync="search"
               @search-change="searchChange"
               @search-reset="searchReset"
               @selection-change="selectionChange"
               @current-change="currentChange"
               @size-change="sizeChange"
               @on-load="onLoad">
      <template slot="menuLeft">
        <el-button type="danger"
                   size="small"
                   icon="el-icon-delete"
                   plain
                   v-if="permission.shift_order_delete"
                   @click="handleDelete">删 除
        </el-button>
        <el-button type="warning"
                   size="small"
                   plain
                   v-if="userInfo.authority.includes('admin')"
                   icon="el-icon-download"
                   @click="handleExport">导出
        </el-button>
      </template>
      <template slot-scope="{row}" slot="menu">
        <el-button
          type="text"
          size="small"
          @click.stop="handleShowDetailList(row)"
        >预约列表
        </el-button>
      </template>
    </avue-crud>
    <el-drawer
      title="预约列表"
      :visible.sync="drawerVisible"
      :direction="direction"
      append-to-body
      :before-close="handleDrawerClose"
      size="1000px"
    >
      <basic-container>
        <avue-crud
          :option="optionDetail"
          :data="dataDetail"
          :page="pageDetail"
          :table-loading="detailLoading"
          ref="crudScope"
          @on-load="onLoadDetail"
        >
        </avue-crud>
      </basic-container>
    </el-drawer>
  </basic-container>
</template>

<script>
  import {getList, getDetail, add, update, remove} from "@/api/shift/shift_order";
  import {getList as getDetailList} from "@/api/shift/shift_order_detail";
  import {mapGetters} from "vuex";
  import {getToken} from "../../util/auth";

  export default {
    data() {
      return {
        drawerVisible: false,
        detailLoading: false,
        selectionListDetail: [],
        pageDetail: {
          pageSize: 10,
          currentPage: 1,
          total: 0,
        },
        dataDetail: [],
        shiftOrderId: 0,
        optionDetail: {
          tip: false,
          searchMenuSpan: 6,
          search:{},
          border: true,
          index: true,
          addBtn: false,
          viewBtn: false,
          editBtn: false,
          delBtn: false,
          menu: false,
          selection: true,
          menuWidth: 200,
          dialogWidth: 900,
          dialogClickModal: false,
          prop: "预约列表",
          column: [
            {
              label: "工号",
              prop: "code"
            },
            {
              label: "姓名",
              prop: "userName"
            },
            {
              label: "联系电话",
              prop: "phone"
            },
            {
              label: "预约区间",
              prop: "intervalName"
            },
            {
              label: "预约时间",
              prop: "startTime"
            },
            {
              label: "状态",
              prop: "statusName"
            },
            {
              label: "备注",
              prop: "remark"
            }
          ],
        },
        form: {},
        query: {},
        loading: true,
        page: {
          pageSize: 10,
          currentPage: 1,
          total: 0
        },
        selectionList: [],
        option: {
          height: 'auto',
          calcHeight: 210,
          searchShow: true,
          searchMenuSpan: 6,
          tip: false,
          border: true,
          editBtn: false,
          delBtn: false,
          index: true,
          viewBtn: false,
          selection: true,
          column: [
            {
              label: "区间",
              prop: "intervalId",
              type: "select",
              dicUrl: "/api/shift/interval/select",
              props: {
                label: "name",
                value: "id"
              },
              search: true,
              rules: [{
                required: true,
                message: "请输入区间id",
                trigger: "blur"
              }]
            },
            {
              label: "日期",
              prop: "date",
              type: "date",
              valueFormat: "yyyy-MM-dd",
              search: true,
              rules: [{
                required: true,
                message: "请输入日期",
                trigger: "blur"
              }]
            },
            {
              label: "开始时间",
              prop: "startTime",
              type: "time",
              valueFormat: "HH:mm:00",
              rules: [{
                required: true,
                message: "请输入开始时间",
                trigger: "blur"
              }]
            },
            {
              label: "预约有效次数",
              prop: "activeOrderCount",
              addDisplay: false
            },
            {
              label: "预约总次数",
              prop: "allOrderCount",
              addDisplay: false
            },
            {
              label: "备注",
              prop: "remark",
              type: "textarea"
            },
          ]
        },
        data: []
      };
    },
    computed: {
      ...mapGetters(["userInfo", "permission"]),
      permissionList() {
        return {
          addBtn: this.vaildData(this.permission.shift_order_add, false),
          viewBtn: this.vaildData(this.permission.shift_order_view, false),
          delBtn: this.vaildData(this.permission.shift_order_delete, false),
          editBtn: this.vaildData(this.permission.shift_order_edit, false)
        };
      },
      ids() {
        let ids = [];
        this.selectionList.forEach(ele => {
          ids.push(ele.id);
        });
        return ids.join(",");
      }
    },
    methods: {
      rowSave(row, done, loading) {
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
      beforeOpen(done, type) {
        if (["edit", "view"].includes(type)) {
          getDetail(this.form.id).then(res => {
            this.form = res.data.data;
          });
        }
        done();
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
      selectionClear() {
        this.selectionList = [];
        this.$refs.crud.toggleSelection();
      },
      currentChange(currentPage){
        this.page.currentPage = currentPage;
      },
      sizeChange(pageSize){
        this.page.pageSize = pageSize;
      },
      onLoad(page, params = {}) {
        this.loading = true;
        getList(page.currentPage, page.pageSize, Object.assign(params, this.query)).then(res => {
          const data = res.data.data;
          this.page.total = data.total;
          this.data = data.records;
          this.loading = false;
          this.selectionClear();
        });
      },
      handleShowDetailList (row) {
        this.drawerVisible = true
        this.shiftOrderId = row.id;
        this.onLoadDetail(this.pageDetail);
      },
      onLoadDetail(page, params = {}) {
        this.joinLoading = true;
        const values = {
          ...params,
          shiftOrderId: this.shiftOrderId,
        };
        getDetailList(
          page.currentPage,
          page.pageSize,
          Object.assign(values, this.query)
        ).then((res) => {
          const data = res.data.data;
          console.log(data);
          this.pageDetail.total = data.total;
          this.dataDetail = data.records;
          this.selectionListDetail = [];
          this.detailLoading = false;
        });
      },
      handleDrawerClose(hide) {
        hide();
      },
      handleExport() {
        this.$confirm("是否导出班车预约数据?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }).then(() => {
          window.open(`/api/shift/shift-order/export?blade-auth=${getToken()}&date=${this.search.date}`);
        });
      }
    }
  };
</script>

<style>
</style>
