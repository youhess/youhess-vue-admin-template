<template>
  <div class="app-container">
    <div class="toTop">
      <el-backtop target=".toTop"></el-backtop>
      <!-- 查询组件 模块 -->
      <div class="block-container">
        <!-- 不加v-bind 也是可以作为prop传值的 -->
        <search-form
          ref="form"
          formSize="mini"
          labelWidth="50px"
          :searchData="searchData"
          :searchForm="searchForm"
          :searchHandle="searchHandle"
        >
        </search-form>
      </div>
      <!-- 表格 模块 -->
      <div class="block-container">
        <!--  -->
        <custom-table
          size="small"
          ref="CustomTable"
          :loading="tableLoading"
          :isPagination="true"
          :isHandle="true"
          :isIndex="true"
          :isBorder="false"
          :tableData="tableData"
          :tableCols="tableCols"
          :pagination="pagination"
          :tableHandles="tableHandles"
          @refresh="getTable"
          :isSelection="true"
          :reserveSection="true"
          @handleSelectionChange="handleSelectionChange"
        >
        </custom-table>
      </div>
      <!-- 表单抽屉 -->
      <div class="drawer_container"></div>
      <!-- 表单弹窗 -->
      <div class="dialog_container">
        <!-- 删除 -->
        <el-dialog
          :title="deleteDialog.title"
          :visible.sync="deleteDialog.visible"
          width="400px"
        >
          <div class="text-center">确定要删除“XXX”吗？</div>
          <span slot="footer" class="dialog-footer">
            <el-button size="small" @click="closeDeleteDialog">
              取消
            </el-button>
            <el-button
              size="small"
              type="primary"
              @click="handleDelete(deleteDialog.row)"
            >
              确定
            </el-button>
          </span>
        </el-dialog>
      </div>
    </div>
  </div>
</template>

<script>
import { getList } from "@/api/table";

// 引入组件
import SearchForm from "@/components/SearchForm";
import DynamicForm from "@/components/DynamicForm";
import CustomTable from "@/components/CustomTable";

export default {
  components: {
    SearchForm,
    CustomTable,
    DynamicForm,
  },
  data() {
    let types = [
      { label: "未知", value: 0 },
      { label: "种类一", value: 1 },
      { label: "种类二", value: 2 },
    ];
    return {
      deleteDialog: { title: "提示", visible: false, row: "" },
      // 搜索=========================================
      searchData: {
        //查询表单的对应的值
        name: "",
        mobile: "",
        idCard: "",
        kindId: "",
        type: "",
        roleIds: [],
        isHasFace: "",
        departmentId: "",
      },
      searchForm: [
        //这里是渲染查询表单的表头和类型的数据
        {
          type: "Input",
          prop: "name",
          width: "200px",
          placeholder: "请输入姓名...",
          size: "small",
          change: this.handleQuery,
        },
        {
          type: "Input",
          prop: "mobile",
          width: "200px",
          placeholder: "请输入手机号...",
          size: "small",
          change: this.handleQuery,
        },
        {
          type: "Input",
          prop: "idCard",
          width: "200px",
          placeholder: "请输入身份证...",
          size: "small",
          change: this.handleQuery,
        },
        {
          type: "Select",
          prop: "kindId",
          width: "200px",
          options: [],
          change: (row) => "",
          placeholder: "请选择工种...",
          size: "small",
          change: this.handleQuery,
        },
        {
          type: "Select",
          prop: "type",
          width: "200px",
          options: types,
          change: (row) => "",
          placeholder: "请选择人员类型...",
          size: "small",
          change: this.handleQuery,
        },
        {
          type: "Select",
          prop: "roleIds",
          width: "200px",
          options: [],
          change: (row) => "",
          placeholder: "请选择角色...",
          size: "small",
          change: this.handleQuery,
        },
      ],
      searchHandle: [
        //重置和查询按钮
        {
          label: "重 置",
          type: "default",
          size: "small",
          handle: this.handleReset,
        },
        {
          label: "查 询",
          type: "primary",
          size: "small",
          handle: this.handleQuery,
        },
      ],
      // 表格============================
      tableData: [],
      tableCols: [
        //表格列数据
        // {
        //   label: "人脸",
        //   prop: "faceUrl",
        //   type: "Image",
        //   class: "w-35px h-35px",
        //   notShowOverflowTooltip: true,
        //   width: "150"
        //   formatter: this.typeToString
        //   isTooltip: true,
        //   formatterTooltip: this.formatterPermissionTooltip
        // },
        { label: "作者", prop: "author" },

        { label: "显示时间", prop: "display_time" },
        { label: "数量", prop: "pageviews" },
        { label: "状态", prop: "status" },
        { label: "简介", prop: "title" },
        {
          label: "操作",
          type: "Button",
          width: "200",
          fixed: "right",
          name: "option",
          btnList: [
            {
              label: "删 除",
              type: "text",
              size: "small",
              handle: this.handleDeleteDialog,
            },
            {
              label: "查 看",
              type: "text",
              size: "small",
              handle: this.handleDetailDrawer,
            },
            {
              label: "编 辑",
              type: "text",
              size: "small",
              handle: this.handleFormDrawer,
            },
            {
              label: "密 码",
              type: "text",
              size: "small",
              handle: this.handlePassWordDrawer,
            },
            // {
            //   label: "删 除",
            //   type: "text",
            //   size: "small",
            //   handle: this.handleDelectDialog,
            //   isPermitted: () =>
            //     this.$btnPermission.isButtonPermission("code-c7")
            // }
          ],
        },
      ],
      tableHandles: [
        //这是表格和表单之间的一个按钮
        {
          label: "新 建",
          type: "primary",
          size: "small",
          handle: this.handleFormDrawer,
        },
        {
          label: "批量选择",
          type: "primary",
          size: "small",
          handle: this.showResignation,
        },
        {
          label: "导 入",
          type: "default",
          size: "small",
          handle: this.onClick_uploadExcel,
        },
        {
          label: "导 出",
          type: "default",
          size: "small",
          handle: this.onClick_download,
          // isPermitted: () => this.$btnPermission.isButtonPermission("code-c14"),
        },
      ],
      tableLoading: false,
      pagination: {
        //分页数据
        pageSize: 10,
        pageNum: 1,
        total: 0,
      },
      // 被选择的items
      selectedItems: [],
    };
  },
  created() {
    this.getTable();
  },
  methods: {
    //获取表格数据
    async getTable() {
      try {
        this.tableLoading = true;
        // let param = {
        //   departmentType: this.departmentType,
        //   ...this.toSearch(this.searchData),
        //   pageNum: this.pagination.pageNum,
        //   pageSize: this.pagination.pageSize,
        // };
        const res = await getList();
        // if (res.success) {
        //   this.tableData = res.data.items || [];
        //   this.pagination.total = parseInt(res.data.total);
        //   console.log("tableData", this.tableData);
        // } else {
        //   this.tableData = [];
        //   this.pagination.total = 0;
        // }
        this.tableData = res.data.items || [];
        this.pagination.total = parseInt(res.data.total);
      } catch (error) {
        console.log("error:", error);
      } finally {
        this.tableLoading = false;
      }
    },
    handleSelectionChange(val) {
      console.log("val", val);
      this.selectedItems = val;
    },
    // 删除弹窗=========================================
    closeDeleteDialog() {
      this.deleteDialog.visible = false;
    },
    // 删除弹窗显示
    handleDeleteDialog(row) {
      console.log("row", row); // 新增按钮undefined 编辑按钮传入对象 删除携带id
      this.deleteDialog.visible = true;
      this.deleteDialog.row = row;
    },
    async handleDelete(row) {
      this.tableLoading = true;
      try {
        // const res = await shipownerRemove({ id: row.id });
        // if (res.success) {
        //   this.$message({
        //     message: `删除成功`,
        //     type: "success",
        //   });
        //   this.deleteDialog.visible = false;
        //   this.getTable();
        // }
        this.deleteDialog.visible = false;
        this.$message({
          message: `删除成功`,
          type: "success",
        });
      } catch (e) {
        console.log(e);
      } finally {
        this.tableLoading = false;
      }
    },
  },
};
</script>
