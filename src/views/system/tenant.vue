<template>
  <basic-container>
    <avue-crud :option="option"
               :data="data"
               ref="crud"
               v-model="form"
               :page="page"
               :permission="permissionList"
               @row-del="rowDel"
               @row-update="rowUpdate"
               @row-save="rowSave"
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
                   v-if="permission.tenant_delete"
                   plain
                   @click="handleDelete">{{$t('route.delete')}}
        </el-button>
      </template>
    </avue-crud>
  </basic-container>
</template>

<script>
  import {getList, remove, update, add} from "@/api/system/tenant";
  import {mapGetters} from "vuex";

  export default {
    data() {
      return {
        form: {},
        selectionList: [],
        query: {},
        page: {
          pageSize: 10,
          currentPage: 1,
          total: 0
        },
        option: {
          tip: false,
          border: true,
          index: true,
          selection: true,
          viewBtn: true,
          dialogWidth: 300,
          dialogHeight: 400,
          column: [
            {
              label: this.$t('tenant.tenantId'),
              prop: "tenantId",
              search: true,
              addDisplay: false,
              editDisplay: false,
              span: 24,
              rules: [{
                required: true,
                message: this.$t('tenant.enterTenantId'),
                trigger: "blur"
              }]
            },
            {
              label: this.$t('tenant.tenantName'),
              prop: "tenantName",
              search: true,
              span: 24,
              rules: [{
                required: true,
                message: this.$t('tenant.enterTenantName'),
                trigger: "blur"
              }]
            },
            {
              label: this.$t('tenant.linkman'),
              prop: "linkman",
              search: true,
              span: 24,
              rules: [{
                required: true,
                message: this.$t('tenant.enterLinkman'),
                trigger: "blur"
              }]
            },
            {
              label: this.$t('tenant.contactNumber'),
              prop: "contactNumber",
              span: 24,
            },
            {
              label: this.$t('tenant.address'),
              prop: "address",
              span: 24,
              minRows: 6,
              type: "textarea",
            }
          ]
        },
        data: []
      };
    },
    computed: {
      ...mapGetters(["permission"]),
      permissionList() {
        return {
          addBtn: this.vaildData(this.permission.tenant_add, false),
          viewBtn: this.vaildData(this.permission.tenant_view, false),
          delBtn: this.vaildData(this.permission.tenant_delete, false),
          editBtn: this.vaildData(this.permission.tenant_edit, false)
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
      rowSave(row, loading, done) {
        add(row).then(() => {
          loading();
          this.onLoad(this.page);
          this.$message({
            type: "success",
            message: this.$t('tenant.success')
          });
        }, error => {
          done();
          console.log(error);
        });
      },
      rowUpdate(row, index, loading, done) {
        update(row).then(() => {
          loading();
          this.onLoad(this.page);
          this.$message({
            type: "success",
            message: this.$t('tenant.success')
          });
        }, error => {
          done();
          console.log(error);
        });
      },
      rowDel(row) {
        this.$confirm(this.$t('tenant.confirmDelete'), {
          confirmButtonText: this.$t('global.confirm'),
          cancelButtonText: this.$t('global.cancel'),
          type: "warning"
        })
          .then(() => {
            return remove(row.id);
          })
          .then(() => {
            this.onLoad(this.page);
            this.$message({
              type: "success",
              message: this.$t('tenant.success')
            });
          });
      },
      searchReset() {
        this.query = {};
        this.onLoad(this.page);
      },
      searchChange(params) {
        this.query = params;
        this.onLoad(this.page, params);
      },
      selectionChange(list) {
        this.selectionList = list;
      },
      handleDelete() {
        if (this.selectionList.length === 0) {
          this.$message.warning(this.$t('tenant.leastOne'));
          return;
        }
        this.$confirm(this.$t('tenant.confirmDelete'), {
          confirmButtonText: this.$t('global.confirm'),
          cancelButtonText: this.$t('global.cancel'),
          type: "warning"
        })
          .then(() => {
            return remove(this.ids);
          })
          .then(() => {
            this.onLoad(this.page);
            this.$message({
              type: "success",
              message: this.$t('tenant.success')
            });
            this.$refs.crud.toggleSelection();
          });
      },
      currentChange(currentPage) {
        this.page.currentPage = currentPage;
      },
      sizeChange(pageSize) {
        this.page.pageSize = pageSize;
      },
      onLoad(page, params = {}) {
        getList(page.currentPage, page.pageSize, Object.assign(params, this.query)).then(res => {
          const data = res.data.data;
          this.page.total = data.total;
          this.data = data.records;
        });
      }
    }
  };
</script>

<style>
</style>
