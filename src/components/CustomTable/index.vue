<!--表格组件 -->
<template>
  <section class="ces-table-page">
    <!-- 表格操作按钮 -->
    <section
      class="ces-handle"
      v-if="isHandle && tableHandles && tableHandles.length > 0"
    >
      <el-button
        v-for="(item, index) in tableHandles"
        v-if="item.isPermitted ? item.isPermitted() : true"
        :size="item.size || size"
        :type="item.type"
        :icon="item.icon"
        @click="item.handle()"
        :key="index"
        :disabled="item.disabled && item.disabled()"
        >{{ item.label }}</el-button
      >
    </section>
    <!-- 数据表格 -->
    <section class="ces-table">
      <el-table
        :header-cell-style="{ background: '#fafafa', color: '#606266' }"
        :data="tableData"
        :size="size"
        :border="isBorder"
        :row-key="rowKey"
        :tree-props="treeProps"
        @select="select"
        @select-all="selectAll"
        @selection-change="handleSelectionChange"
        @row-click="rowClick"
        v-loading="loading"
        header-row-class-name="header_row_style"
        :defaultSelections="defaultSelections"
        :expand-row-keys="expandRowKeys"
        ref="cesTable"
        style="width: 100%"
        :key="tableKey"
        @cell-dblclick="tabClick"
        :row-class-name="tableRowClassName"
        :cell-class-name="tableCellClassName"
      >
        <el-table-column
          v-if="isSelection"
          type="selection"
          :reserve-selection="reserveSection"
          align="center"
          width="50"
        ></el-table-column>
        <el-table-column
          v-if="isIndex"
          type="index"
          :label="indexLabel"
          align="center"
          width="50"
        >
          <template scope="scope">
            <span>{{ scope.$index + 1 }}</span>
          </template>
        </el-table-column>
        <!-- 数据栏 -->
        <el-table-column
          v-for="item in tableCols"
          :key="item.id"
          :prop="item.prop"
          :label="item.label"
          :width="item.width"
          :align="item.align"
          :fixed="item.fixed"
          :show-overflow-tooltip="
            item.notShowOverflowTooltip ? !item.notShowOverflowTooltip : true
          "
          :render-header="item.require ? renderHeader : null"
        >
          <!-- 自定表头 -->
          <template slot="header" v-if="item.isCustomHeader">
            <el-tooltip
              class="item"
              effect="dark"
              :content="item.headerTooltipContent"
              placement="top-start"
            >
              <span>
                <span>{{ item.label }}</span>
                <i class="el-icon-question"> </i>
              </span>
            </el-tooltip>
          </template>
          <template slot-scope="scope">
            <!-- html -->
            <span
              v-if="item.type === 'Html'"
              v-html="item.html(scope.row)"
            ></span>
            <!-- 按钮 -->
            <span v-if="item.type === 'Button'">
              <el-button
                v-for="(btn, index) in item.btnList"
                :disabled="btn.isDisabled && btn.isDisabled(scope.row)"
                v-if="btn.isPermitted ? btn.isPermitted() : true"
                :type="btn.type"
                :size="btn.size || size"
                :icon="btn.icon"
                :style="
                  (btn.handlestyle && btn.handlestyle(scope.row)) || btn.style
                "
                :class="
                  (btn.handleclass && btn.handleclass(scope.row)) || btn.class
                "
                :key="index"
                @click="btn.handle(scope.row)"
                >{{
                  (btn.formatter && btn.formatter(scope.row)) || btn.label
                }}</el-button
              >
            </span>
            <!-- 输入框 -->
            <el-input
              v-if="item.type === 'Input'"
              v-model="scope.row[item.prop]"
              :size="size"
              :disabled="item.isDisabled && item.isDisabled(scope.row)"
              @focus="item.focus && item.focus(scope.row)"
            >
            </el-input>
            <!-- 下拉框 -->
            <el-select
              v-if="item.type === 'Select'"
              v-model="scope.row[item.prop]"
              :size="size"
              :props="item.props"
              :disabled="item.isDisabled && item.isDisabled(scope.row)"
              @change="item.change && item.change(scope.row)"
            >
              <el-option
                v-for="op in item.options"
                :label="op[item.props.label]"
                :value="op[item.props.value]"
                :key="op[item.props.value]"
              ></el-option>
            </el-select>
            <!-- 单选 -->
            <el-radio-group
              v-if="item.type === 'Radio'"
              v-model="scope.row[item.prop]"
              :disabled="item.isDisabled && item.isDisabled(scope.row)"
              @change="item.change && item.change(scope.row)"
            >
              <el-radio
                v-for="(ra, index) in item.radios"
                :label="ra.value"
                :key="index"
                >{{ ra.label }}</el-radio
              >
            </el-radio-group>
            <!-- 复选框 -->
            <el-checkbox-group
              v-if="item.type === 'Checkbox'"
              v-model="scope.row[item.prop]"
              :disabled="item.isDisabled && item.isDisabled(scope.row)"
              @change="item.change && item.change(scope.row)"
            >
              <el-checkbox
                v-for="(ra, index) in item.checkboxs"
                :label="ra.value"
                :key="index"
                >{{ ra.label }}
              </el-checkbox>
            </el-checkbox-group>
            <!-- 评价 -->
            <el-rate
              v-if="item.type === 'Rate'"
              v-model="scope.row[item.prop]"
              :disabled="item.isDisabled && item.isDisabled(scope.row)"
              @change="item.change && item.change(scope.row)"
            >
            </el-rate>
            <!-- 开关 -->
            <el-switch
              v-if="item.type === 'Switch'"
              v-model="scope.row[item.prop]"
              :disabled="item.isDisabled && item.isDisabled(scope.row)"
              :active-value="item.activeValue"
              :inactive-value="item.inactiveValue"
              @change="item.change && item.change(scope.row)"
            ></el-switch>
            <!-- 图像 -->
            <div v-if="item.type === 'Image'">
              <!-- <img
                :src="scope.row[item.prop]"
                @click="item.handle && item.handle(scope.row)"
              /> -->
              <!-- 是数组图片和字符串图片地址的区别 -->
              <el-image
                fit="contain"
                :class="item.class"
                :style="item.style"
                :src="
                  scope.row[item.prop] instanceof Array
                    ? scope.row[item.prop][0]
                    : scope.row[item.prop]
                "
                :preview-src-list="
                  scope.row[item.prop] instanceof Array
                    ? scope.row[item.prop]
                    : [scope.row[item.prop]]
                "
              >
                <div slot="error">
                  <div style="color: rgba(144, 147, 153, 0.8)">未上传</div>
                </div>

                <!-- @click="item.handle && item.handle(scope.row)" -->
              </el-image>
            </div>
            <!-- 滑块 -->
            <el-slider
              v-if="item.type === 'Slider'"
              v-model="scope.row[item.prop]"
              :disabled="item.isDisabled && item.isDisabled(scope.row)"
              @change="item.change && item.change(scope.row)"
            >
            </el-slider>
            <!-- BxTag -->
            <BxTag
              v-if="item.type === 'BxTag'"
              :status="item.status && item.status(scope.row)"
              :txt="item.txt && item.txt(scope.row)"
            ></BxTag>
            <!-- Array -->
            <span
              v-if="item.type === 'Array'"
              :style="item.itemStyle && item.itemStyle(scope.row)"
              :class="item.itemClass && item.item.itemClass(scope.row)"
            >
              <el-tooltip
                placement="top"
                v-if="scope.row[item.prop] && scope.row[item.prop].length > 0"
              >
                <div slot="content">
                  <div v-for="item in scope.row[item.prop]">
                    {{ item }}<br />
                  </div>
                </div>
                <span>
                  {{
                    (item.formatter && item.formatter(scope.row)) ||
                    scope.row[item.prop][0]
                  }}
                </span>
              </el-tooltip>
              <span v-else style="color: rgba(144, 147, 153, 0.8)"> 无 </span>
            </span>
            <!-- 双击input -->
            <span
              v-if="item.type === 'dbClickInput'"
              :style="item.itemStyle && item.itemStyle(scope.row)"
              :class="item.itemClass && item.item.itemClass(scope.row)"
            >
              <!-- <div v-if="scope.row[item.prop]"> -->
              <div
                v-if="
                  scope.row.index === dbClickRowIndex &&
                  scope.column.index === dbClickCellIndex
                "
              >
                <el-input
                  v-focus
                  v-model.number="scope.row[item.prop]"
                  @blur="inputBlur(scope.row[item.prop])"
                  size="mini"
                />
              </div>
              <div v-else class="cursor-pointer">
                {{ scope.row[item.prop] }}
              </div>
              <!-- </div> -->
              <!-- <span v-else style="color:rgba(144,147,153, 0.8)">
                无
              </span> -->
            </span>
            <!-- 默认 -->
            <span
              v-if="!item.type"
              :style="item.itemStyle && item.itemStyle(scope.row)"
              :class="item.itemClass && item.item.itemClass(scope.row)"
            >
              <!-- tooltip -->
              <el-tooltip placement="top" v-if="item.isTooltip">
                <div slot="content">
                  <span
                    v-html="
                      item.formatterTooltip && item.formatterTooltip(scope.row)
                    "
                  ></span>
                </div>
                <!-- mian -->
                <span>
                  <span
                    v-if="
                      (item.formatter && item.formatter(scope.row)) ||
                      scope.row[item.prop]
                    "
                  >
                    {{
                      (item.formatter && item.formatter(scope.row)) ||
                      scope.row[item.prop]
                    }}
                  </span>
                  <span v-else style="color: rgba(144, 147, 153, 0.8)">
                    无
                  </span>
                </span>
              </el-tooltip>
              <!-- none tooltop -->
              <span v-else>
                <span
                  v-if="
                    (item.formatter && item.formatter(scope.row)) ||
                    scope.row[item.prop]
                  "
                >
                  {{
                    (item.formatter && item.formatter(scope.row)) ||
                    scope.row[item.prop]
                  }}
                </span>
                <span v-else style="color: rgba(144, 147, 153, 0.8)"> 无 </span>
              </span>
            </span>
          </template>
        </el-table-column>
      </el-table>
    </section>
    <!-- 分页 -->
    <section class="ces-pagination" v-if="isPagination">
      <el-pagination
        style="
          display: flex;
          justify-content: center;
          height: 100%;
          align-items: center;
          margin-bottom: 24px;
        "
        @current-change="handleCurrentChange"
        @size-change="handleSizeChange"
        layout="total,sizes ,prev, pager, next,jumper"
        :page-size="pagination.pageSize"
        :current-page="pagination.pageNum"
        :total="pagination.total"
      ></el-pagination>
    </section>
  </section>
</template>

<script>
import BxTag from "@/components/BxTag";
export default {
  name: "CustomTable",
  components: {
    BxTag,
  },
  directives: {
    focus: {
      // 指令的定义
      inserted: function (el) {
        el.getElementsByTagName("input")[0].focus();
        // el.querySelector("input").focus();
      },
    },
  },
  props: {
    // 表格型号：mini,medium,small
    size: { type: String, default: "medium" },
    isBorder: { type: Boolean, default: true },
    loading: { type: Boolean, default: false },
    tableKey: { type: Number, default: 0 },
    // 树形结构设置
    treeProps: {
      type: Object,
      default: () => ({ children: "children", hasChildren: "hasChildren" }),
    },
    rowKey: {
      type: String,
      default: "id",
    },
    // 表格操作
    isHandle: { type: Boolean, default: false },
    tableHandles: { type: Array, default: () => [] },
    expandRowKeys: { type: Array, default: () => [] },
    // 表格数据
    tableData: { type: Array, default: () => [] },
    // 表格列配置
    tableCols: { type: Array, default: () => [] },
    // 是否显示表格复选框
    isSelection: { type: Boolean, default: false },
    // 储备
    reserveSection: { type: Boolean, default: false },
    defaultSelections: { type: [Array, Object], default: () => null },
    // 是否显示表格索引
    isIndex: { type: Boolean, default: false },
    indexLabel: { type: String, default: "序号" },
    // 是否显示分页
    isPagination: { type: Boolean, default: true },
    // 分页数据
    pagination: {
      type: Object,
      default: () => ({ pageSize: 10, pageNum: 1, total: 0 }),
    },
  },
  data() {
    return {
      dbClickRowIndex: null, // 当前点击的行索引
      dbClickCellIndex: null, // 当前点击的列索引
      tabClickLabel: "", // 当前点击的列名
    };
  },
  watch: {
    defaultSelections(val) {
      this.$nextTick(function () {
        if (Array.isArray(val)) {
          val.forEach((row) => {
            this.$refs.cesTable.toggleRowSelection(row);
          });
        } else {
          this.$refs.cesTable.toggleRowSelection(val);
        }
      });
    },
    // tableKey(val) {
    //   console.log("val", val);
    // }
  },
  methods: {
    // 控制input显示 row 当前行 column 当前列
    tabClick(row, column, cell, event) {
      console.log("row:", row);
      // console.log("column:", column);
      // console.log("column.index:", column.index);
      // console.log("column.label:", column.label);
      this.dbClickRowIndex = row.index;
      this.dbClickCellIndex = column.index;
      this.tabClickLabel = column.label;
    },
    // 失去焦点初始化
    inputBlur() {
      // console.log("this.tableData:", this.tableData[this.dbClickRowIndex]);
      // TODO 向外抛出 失焦行信息
      this.$emit("inputBlur", this.tableData[this.dbClickRowIndex]);
      this.dbClickRowIndex = null;
      this.dbClickCellIndex = null;
      this.tabClickLabel = "";
    },
    // 把每一行的索引放进row
    tableRowClassName({ row, rowIndex }) {
      row.index = rowIndex;
    },

    // 把每一列的索引放进column
    tableCellClassName({ column, columnIndex }) {
      column.index = columnIndex;
    },
    // 表格勾选
    select(rows, row) {
      this.$emit("select", rows, row);
    },
    // 全选
    selectAll(rows) {
      this.$emit("select", rows);
    },
    handleSelectionChange(val) {
      this.$emit("handleSelectionChange", val);
    },
    rowClick(val) {
      this.$emit("rowClick", val);
    },
    //
    handleCurrentChange(val) {
      this.pagination.pageNum = val;
      this.$emit("refresh");
    },
    handleSizeChange(val) {
      this.pagination.pageSize = val;
      this.$emit("refresh");
    },
    clearSelectedItems() {
      this.$refs.cesTable.clearSelection();
    },
    // tableRowClassName({rowIndex}) {
    //     if (rowIndex % 2 === 0) {
    //         return "stripe-row";
    //     }
    //     return "";
    // }
    renderHeader(h, obj) {
      return h("span", { class: "ces-table-require" }, obj.column.label);
    },
    monitoring() {
      // 监听事件
      this.$on("toggleRowSelection", (res) => {
        // console.log("方法1:触发监听事件监听成功");
        // console.log("res", res);
        this.$refs.cesTable.toggleRowSelection(res.row, res.isSelected);
      });
      // 监听事件
      this.$on("clearSelection", (res) => {
        // console.log("方法1:触发监听事件监听成功");
        // console.log("res", res);
        this.$refs.cesTable.clearSelection();
      });
    },
  },
  mounted() {
    this.monitoring(); // 注册监听事件
  },
};
</script>
<style lang="scss">
.ces-table-require::before {
  content: "*";
  color: red;
}

.el-pagination {
  float: right;
  margin-right: 10px;
  margin-top: 5px;
}

.ces-handle {
  margin-bottom: 24px;
}

/* 修复loding居左的问题 */
.el-loading-spinner {
  left: calc(50% - 25px);
}
</style>
