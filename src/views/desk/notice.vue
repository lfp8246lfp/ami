<template>
  <basic-container>
    <avue-crud :option="option"
               :data="data"
               :page="page"
               @row-del="rowDel"
               v-model="form"
               :permission="permissionList"
               @row-update="rowUpdate"
               @row-save="rowSave"
               :before-open="beforeOpen"
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
                   v-if="permission.notice_delete"
                   @click="handleDelete">{{$t('route.delete')}}
        </el-button>
      </template>
      <template slot-scope="{row}"
                slot="category">
        <el-tag>{{row.categoryName}}</el-tag>
      </template>
    </avue-crud>
  </basic-container>
</template>

<script>
  import {getList, remove, update, add, getNotice} from "@/api/dept/notice";
  import {mapGetters} from "vuex";

  export default {
    data() {
      return {
        form: {},
        query: {},
        page: {
          pageSize: 10,
          currentPage: 1,
          total: 0
        },
        selectionList: [],
        option: {
          tip: false,
          border: true,
          index: true,
          viewBtn: true,
          selection: true,
          column: [
            {
              label: this.$t('notice.title'),
              prop: "title",
              row: true,
              search: true,
              rules: [{
                required: true,
                message: this.$t('notice.enterTitle'),
                trigger: "blur"
              }]
            },
            {
              label: this.$t('notice.category'),
              type: "select",
              row: true,
              dicUrl: "/api/ami-system/dict/dictionary?code=notice",
              props: {
                label: "dictValue",
                value: "dictKey"
              },
              slot: true,
              prop: "category",
              search: true,
              rules: [{
                required: true,
                message: this.$t('notice.enterSelect'),
                trigger: "blur"
              }]
            },
            {
              label: this.$t('notice.releaseTime'),
              prop: "releaseTime",
              type: "date",
              format: "yyyy-MM-dd hh:mm:ss",
              valueFormat: "yyyy-MM-dd hh:mm:ss",
              rules: [{
                required: true,
                message: this.$t('notice.enterReleaseTime'),
                trigger: "blur"
              }]
            },
            {
              label: this.$t('notice.content'),
              prop: "content",
              span: 24,
              minRows: 6,
              type: "textarea"
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
          addBtn: this.vaildData(this.permission.notice_add, false),
          viewBtn: this.vaildData(this.permission.notice_view, false),
          delBtn: this.vaildData(this.permission.notice_delete, false),
          editBtn: this.vaildData(this.permission.notice_edit, false)
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
            message: this.$t('notice.success')
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
            message: this.$t('notice.success')
          });
        }, error => {
          done();
          console.log(error);
        });
      },
      rowDel(row) {
        this.$confirm(this.$t('notice.confirmDelete'), {
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
              message: this.$t('notice.success')
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
          this.$message.warning(this.$t('notice.leastOne'));
          return;
        }
        this.$confirm(this.$t('notice.confirmDelete'), {
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
              message: this.$t('notice.success')
            });
            this.$refs.crud.toggleSelection();
          });
      },
      beforeOpen(done, type) {
        if (["edit", "view"].includes(type)) {
          getNotice(this.form.id).then(res => {
            this.form = res.data.data;
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
