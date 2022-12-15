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
          itemSize="small"
          labelWidth="120px"
          :clearable="true"
          v-model="searchData"
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
          :isIndex="true"
          :isBorder="false"
          :tableData="tableData"
          :tableCols="tableCols"
          :pagination="pagination"
          :isHandle="true"
          :tableHandles="tableHandles"
          :isFilterColumn="true"
          :filterColumnDisabledArr="['操作']"
          @refresh="getTable"
          :isSelection="false"
          :reserveSection="true"
          @select="selection"
          @selectionChange="selectionChange"
          @rowClick="rowClick"
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
    let selectItems = [
      { label: "未知", value: 0 },
      { label: "种类一", value: 1 },
      { label: "种类二", value: 2 },
    ];
    let radioItems = [
      { label: "未知", value: 0 },
      { label: "一", value: 1 },
      { label: "二", value: 2 },
    ];
    let radioButtonItems = [
      { label: "未知", value: 0 },
      { label: "一", value: 1 },
      { label: "二", value: 2 },
    ];
    let checkboxItems = [
      { label: "未知", value: 0 },
      { label: "一", value: 1 },
      { label: "二", value: 2 },
    ];
    return {
      deleteDialog: { title: "提示", visible: false, row: "" },
      // 搜索=========================================
      searchData: {
        //查询表单的对应的值
        text: "",
        kindId: "",
        radio: 0,
        radiobutton: 0,
        slider: 0,
        radiobutton: 0,
        checkbox: [],
        switch: true,
        date: "",
        time: "",
        dateTime: "",
        datetimerange: [],
      },
      searchForm: [
        {
          label: "输入框",
          type: "Input",
          prop: "text",
          width: "200px",
          placeholder: "请输入文本",
          change: this.handleQuery,
        },
        {
          label: "下拉框",
          type: "Select",
          prop: "selectItem",
          width: "200px",
          options: selectItems,
          change: this.handleQuery,
          placeholder: "请输入下拉选择项",
          multiple: true,
          reserveKeyword: true,
          filterable: true,
        },
        {
          label: "单选框",
          type: "Radio",
          prop: "radio",
          width: "200px",
          change: this.handleQuery,
          radios: radioItems,
        },
        {
          label: "单选框按钮",
          type: "RadioButton",
          prop: "radiobutton",
          width: "200px",
          change: this.handleQuery,
          radios: radioButtonItems,
        },
        {
          label: "复选框",
          type: "Checkbox",
          prop: "checkbox",
          width: "200px",
          change: this.handleQuery,
          checkboxs: checkboxItems,
        },
        {
          label: "日期",
          type: "Date",
          prop: "date",
          width: "200px",
          placeholder: "请选择日期",
          change: this.handleQuery,
          pickerOptions: {},
        },
        {
          label: "时间",
          type: "Time",
          prop: "time",
          width: "200px",
          placeholder: "请选择时间",
          change: this.handleQuery,
          pickerOptions: {},
        },
        {
          label: "滑块",
          type: "Slider",
          prop: "slider",
          width: "200px",
          change: this.handleQuery,
        },
        {
          label: "开关",
          type: "Switch",
          prop: "switch",
          width: "200px",
          change: this.handleQuery,
        },
        {
          label: "日期时间",
          type: "DateTime",
          prop: "dateTime",
          placeholder: "请选择日期时间",
          width: "200px",
          change: this.handleQuery,
          pickerOptions: {},
        },
        {
          label: "日期时间选择器",
          type: "Datetimerange",
          prop: "datetimerange",
          startPlaceholder: "请输入开始日期时间",
          endPlaceholder: "请输入截止时间",
          valueFormat: "yyyy-MM-dd HH:mm:ss",
          change: this.handleQuery,
          pickerOptions: {},
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
          width: "100",
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
              handle: (row) => {},
            },
          ],
        },
      ],
      tableHandles: [
        //这是表格和表单之间的一个按钮
        {
          label: "新 建",
          type: "primary",
          size: "small",
          handle: (row) => {},
        },
        {
          label: "批量选择",
          type: "primary",
          size: "small",
          handle: (row) => {},
        },
        {
          label: "导 入",
          type: "default",
          size: "small",
          handle: (row) => {},
        },
        {
          label: "导 出",
          type: "default",
          size: "small",
          handle: (row) => {},
          // isShown: () => this.$btnPermission.isButtonPermission("code-c14"),
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
    // 表格组件-----------------------------------------------
    //  表格选择
    selection(rows, row) {
      console.log(" @select:被选中项 rows", rows);
      console.log(" @select:被选中当前项 row", row);
    },
    selectionChange(val) {
      console.log("@selectionChange:被选中项 rows", val);
      this.selectedItems = val;
    },
    //  表格单击
    rowClick(row) {
      console.log("@rowClick:被单击选项", row);
    },
    //搜索组件-------------------------------------------------
    handleReset() {
      for (var key in this.searchData) {
        if (typeof this.searchData[key] == "number") {
          this.$set(this.searchData, key, 0);
        } else if (typeof this.searchData[key] == "boolean") {
          this.$set(this.searchData, key, 0);
        } else if (Array.isArray(this.searchData[key])) {
          this.$set(this.searchData, key, []);
        } else {
          this.$set(this.searchData, key, "");
        }
      }
      this.getTable();
    },
    // 表格搜索
    handleQuery() {
      console.log("查询！", this.searchData);
      this.getTable();
    },
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
