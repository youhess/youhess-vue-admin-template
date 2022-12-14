<template>
  <div class="search-form">
    <el-form
      ref="refForm"
      class="form"
      :size="formSize"
      inline
      :label-width="labelWidth"
    >
      <el-form-item
        v-for="item in searchForm"
        :key="item.prop"
        :label="item.label"
        class="form-item"
      >
        <!-- 输入框 -->
        <el-input
          v-if="item.type === 'Input'"
          v-model="sonSearchData[item.prop]"
          :style="{ width: item.width }"
          :size="itemSize ? itemSize : item.size"
          :placeholder="item.placeholder"
          :clearable="clearable"
          @change="item.change && item.change(sonSearchData[item.prop])"
        />
        <!-- 下拉框 -->
        <el-select
          v-if="item.type === 'Select'"
          v-model="sonSearchData[item.prop]"
          :size="itemSize ? itemSize : item.size"
          :style="{ width: item.width }"
          :placeholder="item.placeholder"
          :filterable="item.filterable"
          :multiple="item.multiple"
          :remote="item.remote"
          :reserve-keyword="item.reserveKeyword"
          :clearable="clearable"
          :remote-method="
            item.remoteMethod && item.remoteMethod(sonSearchData[item.prop])
          "
          @change="item.change && item.change(sonSearchData[item.prop])"
        >
          <el-option
            v-for="op in item.options"
            :key="op.value"
            :label="op.label"
            :value="op.value"
          />
        </el-select>
        <!-- 单选 -->
        <el-radio-group
          v-if="item.type === 'Radio'"
          v-model="sonSearchData[item.prop]"
          :size="itemSize ? itemSize : item.size"
          :style="{ width: item.width }"
          @change="item.change && item.change(sonSearchData[item.prop])"
        >
          <el-radio
            v-for="ra in item.radios"
            :key="ra.value"
            :label="ra.value"
          >{{ ra.label }}</el-radio>
        </el-radio-group>
        <!-- 单选按钮 -->
        <el-radio-group
          v-if="item.type === 'RadioButton'"
          v-model="sonSearchData[item.prop]"
          :size="itemSize ? itemSize : item.size"
          :style="{ width: item.width }"
          @change="item.change && item.change(sonSearchData[item.prop])"
        >
          <el-radio-button
            v-for="ra in item.radios"
            :key="ra.value"
            :label="ra.value"
          >{{ ra.label }}</el-radio-button>
        </el-radio-group>
        <!-- 复选框 -->
        <el-checkbox-group
          v-if="item.type === 'Checkbox'"
          v-model="sonSearchData[item.prop]"
          :size="itemSize ? itemSize : item.size"
          :clearable="clearable"
          :style="{ width: item.width }"
          @change="item.change && item.change(sonSearchData[item.prop])"
        >
          <el-checkbox
            v-for="ch in item.checkboxs"
            :key="ch.value"
            :label="ch.value"
          >{{ ch.label }}</el-checkbox>
        </el-checkbox-group>
        <!-- 日期 -->
        <el-date-picker
          v-if="item.type === 'Date'"
          v-model="sonSearchData[item.prop]"
          :placeholder="item.placeholder"
          :size="itemSize ? itemSize : item.size"
          type="date"
          :style="{ width: item.width }"
          :picker-options="item.pickerOptions"
          @change="item.change && item.change(sonSearchData[item.prop])"
        />
        <!-- 时间 -->
        <el-time-picker
          v-if="item.type === 'Time'"
          v-model="sonSearchData[item.prop]"
          :size="itemSize ? itemSize : item.size"
          :picker-options="item.pickerOptions"
          :placeholder="item.placeholder"
          :style="{ width: item.width }"
          @change="item.change && item.change(sonSearchData[item.prop])"
        />
        <!-- 日期时间 -->
        <el-date-picker
          v-if="item.type === 'DateTime'"
          v-model="sonSearchData[item.prop]"
          type="datetime"
          :placeholder="item.placeholder"
          :disabled="item.disable && item.disable(sonSearchData[item.prop])"
          :picker-options="item.pickerOptions"
          :size="itemSize ? itemSize : item.size"
          :style="{ width: item.width }"
          @change="item.change && item.change(sonSearchData[item.prop])"
        />
        <!-- 日期范围 -->
        <el-date-picker
          v-if="item.type === 'Datetimerange'"
          v-model="sonSearchData[item.prop]"
          type="datetimerange"
          range-separator="~"
          :start-placeholder="item.startPlaceholder"
          :end-placeholder="item.endPlaceholder"
          :value-format="item.valueFormat"
          :picker-options="item.pickerOptions"
          :size="itemSize ? itemSize : item.size"
          @change="item.change && item.change(sonSearchData[item.prop])"
        />
        <!-- 滑块 -->
        <el-slider
          v-if="item.type === 'Slider'"
          v-model="sonSearchData[item.prop]"
          :size="itemSize ? itemSize : item.size"
          :style="{ width: item.width }"
          @change="item.change && item.change(sonSearchData[item.prop])"
        />
        <!-- 开关 -->
        <div v-if="item.type === 'Switch'" :style="{ width: item.width }">
          <el-switch
            v-model="sonSearchData[item.prop]"
            :size="itemSize ? itemSize : item.size"
            @change="item.change && item.change(sonSearchData[item.prop])"
          />
        </div>
      </el-form-item>
      <template v-if="isHandle">
        <el-form-item
          v-for="(item, index) in searchHandle"
          :key="index"
          inline
          class="form-item"
        >
          <el-button
            v-if="item.isPermitted ? item.isPermitted() : true"
            :type="item.type"
            :size="itemSize ? itemSize : item.size || formSize"
            @click="item.handle()"
          >{{ item.label }}</el-button>
        </el-form-item>
      </template>

    </el-form>
  </div>
</template>

<script>
export default {
  name: "SearchForm",
  model: {
    // 需要双向绑定的 props 变量名称，也就是父组件通过 v-model 与子组件双向绑定的变量
    prop: "searchData",
    // 定义由 $emit 传递变量的名称
    event: "searchDataChange",
  },
  props: {
    isHandle: {
      type: Boolean,
      default: true,
    },
    labelWidth: {
      type: String,
      default: "50px",
    },
    // 表单大小设置
    formSize: {
      type: String,
      default: "mini",
    },
    itemSize: {
      type: String,
      default: "",
    },
    searchForm: {
      type: Array,
      default: () => [],
    },
    searchHandle: {
      type: Array,
      default: () => [],
    },
    searchData: {
      type: Object,
      default: () => {},
    },
    clearable: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      // 子组件不能修改 props 下的变量，所以定义一个临时变量
      sonSearchData: this.searchData,
    };
  },
  watch: {
    sonSearchData: {
      // 这里箭头函数指向可能会出现问题
      handler: function(newVal, oldVal) {
        this.$emit("searchDataChange", this.sonSearchData);
      },
      deep: true,
    },
  },
  methods: {},
};
</script>
<style lang="scss">
.search-form {
  display: flex;
  align-items: center;
  .form {
    .form-item {
      // .el-form-item__label {
      //   width: 80px !important;
      // }
      margin-right: 12px;
    }
  }
}
</style>
