<template>
  <basic-container>
    <avue-crud
      ref="crud"
      v-model="form"
      :option="option"
      :data="nlpModelList"
      :page.sync="page"
      :table-loading="loading"
      :permission="permissionList"
      @row-save="rowSave"
      @row-update="rowUpdate"
      @row-del="rowDel"
      @on-load="onLoad"
      @search-change="searchChange"
      @search-reset="searchReset"
      @selection-change="selectionChange"
      @current-change="currentChange"
      @size-change="sizeChange"
      @refresh-change="refreshChange"
    >
      <template v-slot:menu="scope">
        <el-button
          type="text"
          icon="el-icon-edit"
          size="small"
          @click.stop="handleOperation(scope.row, 'edit')"
        >
          编辑
        </el-button>
        <el-button
          type="text"
          icon="el-icon-delete"
          size="small"
          @click.stop="handleOperation(scope.row, 'delete')"
        >
          删除
        </el-button>
      </template>
    </avue-crud>
  </basic-container>
</template>

<script>
import {
  getModelRelationshipList,
  addModelRelationship,
  editModelRelationship,
  deleteModelRelationship,
} from "@/api/modelManger";

export default {
  name: "ModelRelationship",
  data() {
    return {
      loading: false,
      nlpModelList: [],
      page: {
        current: 1,
        size: 10,
        total: 0,
      },
      form: {},
      option: {
        stripe: true,
        border: true,
        labelWidth: 120,
        searchMenuSpan: 8,
        column: [
          {
            label: "ID",
            prop: "id",
            width: 80,
            addDisplay: false,
            editDisplay: false,
            viewDisplay: false,
          },
          { label: "租户ID", prop: "tenantId", search: true, span: 8 },
          { label: "源库名", prop: "srcSchema", search: true, span: 8 },
          { label: "源表名", prop: "srcTableName", search: true, span: 8 },
          { label: "主键", prop: "primaryKey", span: 8 },
          {
            label: "源查询条件",
            prop: "srcWhere",
            type: "textarea",
            minRows: 2,
            span: 24,
          },
          { label: "目标库名", prop: "destSchema", span: 8 },
          { label: "目标表名", prop: "destTableName", span: 8 },
          { label: "模型ID", prop: "varModelId", span: 8 },
        ],
      },
    };
  },
  computed: {
    permissionList() {
      // 根据实际权限实现返回相应权限对象
      return (
        this.$store?.state?.permission?.modelRelationship || {}
      );
    },
  },
  methods: {
    /**
     * 表格数据加载（分页、查询）
     * @param {Object} pageInfo avue-crud 当前分页信息
     * @param {Object} params   查询条件
     */
    async onLoad(pageInfo, params) {
      this.loading = true;
      try {
        const query = {
          ...params,
          current: pageInfo.currentPage,
          size: pageInfo.pageSize,
        };
        const { data } = await getModelRelationshipList(query);
        this.page.current = data.current || 1;
        this.page.size = data.size || query.size;
        this.page.total = data.total || 0;
        this.nlpModelList = data.records || [];
      } catch (err) {
        console.error(err);
      } finally {
        this.loading = false;
      }
    },

    /** 新增保存 */
    async rowSave(row, done, loading) {
      try {
        await addModelRelationship(row);
        this.$message.success("新增成功");
        done();
        this.$refs.crud.refresh();
      } catch (err) {
        loading();
      }
    },

    /** 编辑保存 */
    async rowUpdate(row, index, done, loading) {
      try {
        await editModelRelationship(row);
        this.$message.success("修改成功");
        done();
        this.$refs.crud.refresh();
      } catch (err) {
        loading();
      }
    },

    /** 删除 */
    async rowDel(row, index, done) {
      try {
        await this.$confirm("确定删除该数据吗？", "提示", { type: "warning" });
        await deleteModelRelationship({ id: row.id });
        this.$message.success("删除成功");
        done();
        this.$refs.crud.refresh();
      } catch (err) {
        done();
      }
    },

    /** 自定义操作按钮 */
    handleOperation(row, type) {
      if (type === "edit") {
        this.$refs.crud.rowEdit(row);
      } else if (type === "delete") {
        this.$refs.crud.rowDel(row);
      }
    },

    // 搜索变更
    searchChange(params, done) {
      this.page.current = 1;
      this.onLoad(this.page, params);
      done();
    },

    // 搜索重置
    searchReset(done) {
      this.page.current = 1;
      this.onLoad(this.page, {});
      done();
    },

    // 选中行变更
    selectionChange(selection) {
      this.selectedRows = selection;
    },

    // 页码切换
    currentChange(current) {
      this.page.current = current;
    },

    // 每页条数切换
    sizeChange(size) {
      this.page.size = size;
    },

    // 刷新按钮
    refreshChange() {
      this.onLoad(this.page, this.searchParams || {});
    },

    // 弹窗打开前
    beforeOpen(done) {
      done();
    },
  },
};
</script>

<style scoped></style>