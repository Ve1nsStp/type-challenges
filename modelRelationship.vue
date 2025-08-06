<template>
  <basic-container>
    <avue-crud :option="option" :table-loading="loading" :data="nlpModelList" :page.sync="page"
      :permission="permissionList" v-model="form" ref="crud" @row-update="rowUpdate" @row-save="rowSave"
      @row-del="rowDel" :before-open="beforeOpen" @search-change="searchChange" @search-reset="searchReset"
      @selection-change="selectionChange" @current-change="currentChange" @size-change="sizeChange"
      @refresh-change="refreshChange" @on-load="onLoad">
      <template slot-scope="scope" slot="menu">
        <el-button type="text" icon="el-icon-edit" size="small" @click.stop="handleOperation(scope.row, 'edit')">编辑
        </el-button>
        <el-button type="text" icon="el-icon-delete" size="small" @click.stop="handleOperation(scope.row, 'delete')">删除
        </el-button>
      </template>
    </avue-crud>
  </basic-container>
</template>

<script>
import { getModelRelationshipList, addModelRelationship, editModelRelationship, deleteModelRelationship } from "@/api/modelManger";

export default {
  name: "modelRelationship",
  data() {
    return {
      form: {},
      page: {
        pageSize: 10,
        currentPage: 1,
        total: 0
      },
      loading: false,
      nlpModelList: [],
      permissionList: {
        addBtn: true,
        editBtn: true,
        delBtn: true,
        viewBtn: false
      },
      option: {
        height: 'auto',
        calcHeight: 30,
        tip: false,
        searchShow: true,
        searchMenuSpan: 6,
        border: true,
        index: true,
        viewBtn: false,
        selection: true,
        dialogClickModal: false,
        column: [
          {
            label: "ID",
            prop: "id",
            width: 80,
            hide: true,
            addDisplay: false,
            editDisplay: false
          },
          {
            label: "租户ID",
            prop: "tenantId",
            search: true,
            width: 120,
            rules: [{
              required: true,
              message: "请输入租户ID",
              trigger: "blur"
            }]
          },
          {
            label: "源数据库",
            prop: "srcSchema",
            search: true,
            width: 120,
            rules: [{
              required: true,
              message: "请输入源数据库",
              trigger: "blur"
            }]
          },
          {
            label: "源表名",
            prop: "srcTableName",
            search: true,
            width: 150,
            rules: [{
              required: true,
              message: "请输入源表名",
              trigger: "blur"
            }]
          },
          {
            label: "主键",
            prop: "primaryKey",
            width: 120,
            rules: [{
              required: true,
              message: "请输入主键",
              trigger: "blur"
            }]
          },
          {
            label: "源条件",
            prop: "srcWhere",
            width: 200,
            type: "textarea",
            span: 24,
            placeholder: "请输入SQL WHERE条件"
          },
          {
            label: "目标数据库",
            prop: "destSchema",
            search: true,
            width: 120,
            rules: [{
              required: true,
              message: "请输入目标数据库",
              trigger: "blur"
            }]
          },
          {
            label: "目标表名",
            prop: "destTableName",
            search: true,
            width: 150,
            rules: [{
              required: true,
              message: "请输入目标表名",
              trigger: "blur"
            }]
          },
          {
            label: "变量模型ID",
            prop: "varModelId",
            width: 120,
            type: "number",
            rules: [{
              required: true,
              message: "请输入变量模型ID",
              trigger: "blur"
            }]
          }
        ]
      }
    };
  },
  created() {
    this.getList();
  },
  methods: {
    // 获取列表数据
    getList() {
      this.loading = true;
      const params = {
        current: this.page.currentPage,
        size: this.page.pageSize,
        ...this.searchForm
      };
      
      getModelRelationshipList(params).then(res => {
        if (res.data.success) {
          this.nlpModelList = res.data.data.records || [];
          this.page.total = res.data.data.total || 0;
        }
        this.loading = false;
      }).catch(() => {
        this.loading = false;
      });
    },
    
    // 页面刷新
    onLoad(page, params = {}) {
      this.page.currentPage = page.currentPage;
      this.page.pageSize = page.pageSize;
      this.getList();
    },
    
    // 刷新按钮
    refreshChange() {
      this.getList();
    },
    
    // 页码改变
    currentChange(currentPage) {
      this.page.currentPage = currentPage;
      this.getList();
    },
    
    // 页数改变
    sizeChange(pageSize) {
      this.page.pageSize = pageSize;
      this.getList();
    },
    
    // 搜索
    searchChange(params, done) {
      this.searchForm = params;
      this.page.currentPage = 1;
      this.getList();
      done();
    },
    
    // 搜索重置
    searchReset() {
      this.searchForm = {};
      this.page.currentPage = 1;
      this.getList();
    },
    
    // 选择改变
    selectionChange(list) {
      this.selectionList = list;
    },
    
    // 新增
    rowSave(row, done, loading) {
      addModelRelationship(row).then(() => {
        this.$message.success("新增成功");
        done();
        this.getList();
      }).catch(() => {
        loading();
      });
    },
    
    // 编辑
    rowUpdate(row, index, done, loading) {
      editModelRelationship(row).then(() => {
        this.$message.success("修改成功");
        done();
        this.getList();
      }).catch(() => {
        loading();
      });
    },
    
    // 删除
    rowDel(row) {
      this.$confirm('确定将选择数据删除?', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        return deleteModelRelationship(row.id);
      }).then(() => {
        this.$message.success("删除成功");
        this.getList();
      });
    },
    
    // 打开弹窗前的回调
    beforeOpen(done, type) {
      if (["edit", "view"].includes(type)) {
        // 编辑或查看时可以做一些数据处理
      }
      done();
    },
    
    // 自定义操作
    handleOperation(row, type) {
      if (type === 'edit') {
        this.$refs.crud.rowEdit(row);
      } else if (type === 'delete') {
        this.rowDel(row);
      }
    }
  }
};
</script>

<style scoped>
</style>