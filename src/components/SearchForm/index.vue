<template>
  <div class="ces-search">
    <el-form class="form" ref="refForm" :size="formSize" inline :label-width="labelWidth">
      <el-form-item v-for="item in searchForm" :label="item.label" :key="item.prop" class="formItem">
        <!-- 输入框 -->
        <el-input v-if="item.type === 'Input'" v-model="searchData[item.prop]" :style="{ width: item.width }"
          :size="item.size" :placeholder="item.placeholder" @change="item.change && item.change(searchData[item.prop])"
          clearable></el-input>
        <!-- 下拉框 -->
        <el-select v-if="item.type === 'Select'" v-model="searchData[item.prop]"
          @change="item.change && item.change(searchData[item.prop])" :size="item.size" :style="{ width: item.width }"
          :placeholder="item.placeholder" :filterable="item.filterable" :multiple="item.multiple" :remote="item.remote"
          :reserve-keyword="item.reserveKeyword" clearable :remote-method="
            item.remoteMethod && item.remoteMethod(searchData[item.prop])
          ">
          <el-option v-for="op in item.options" :label="op.label" :value="op.value" :key="op.value"></el-option>
        </el-select>
        <!-- 单选 -->
        <el-radio-group v-if="item.type === 'Radio'" @change="item.change && item.change(searchData[item.prop])"
          v-model="searchData[item.prop]" :size="item.size">
          <el-radio v-for="ra in item.radios" :label="ra.value" :key="ra.value">{{ ra.label }}</el-radio>
        </el-radio-group>
        <!-- 单选按钮 -->
        <el-radio-group v-if="item.type === 'RadioButton'" v-model="searchData[item.prop]" :size="item.size"
          @change="item.change && item.change(searchData[item.prop])">
          <el-radio-button v-for="ra in item.radios" :label="ra.value" :key="ra.value">{{ ra.label }}</el-radio-button>
        </el-radio-group>
        <!-- 复选框 -->
        <el-checkbox-group v-if="item.type === 'Checkbox'" v-model="searchData[item.prop]"
          @change="item.change && item.change(searchData[item.prop])" :size="item.size" clearable>
          <el-checkbox v-for="ch in item.checkboxs" :label="ch.value" :key="ch.value">{{ ch.label }}</el-checkbox>
        </el-checkbox-group>
        <!-- 日期 -->
        <el-date-picker v-if="item.type === 'Date'" v-model="searchData[item.prop]"
          @change="item.change && item.change(searchData[item.prop])" :size="item.size"></el-date-picker>
        <!-- 时间 -->
        <el-time-select v-if="item.type === 'Time'" v-model="searchData[item.prop]"
          @change="item.change && item.change(searchData[item.prop])" type="" :size="item.size">
        </el-time-select>
        <!-- 日期时间 -->
        <el-date-picker v-if="item.type === 'DateTime'" type="datetime" v-model="searchData[item.prop]"
          @change="item.change && item.change(searchData[item.prop])"
          :disabled="item.disable && item.disable(searchData[item.prop])" :size="item.size"></el-date-picker>
        <!-- 日期范围 -->
        <el-date-picker v-if="item.type === 'datetimerange'" type="datetimerange" v-model="searchData[item.prop]"
          range-separator="~" :start-placeholder="item.startPlaceholder" :end-placeholder="item.endPlaceholder"
          :value-format="item.valueFormat" :picker-options="item.pickerOptions" :size="item.size"
          @change="item.change && item.change(searchData[item.prop])">
        </el-date-picker>
        <!-- 滑块 -->
        <!-- <el-slider v-if="item.type==='Slider'" v-model="searchData[item.prop]"></el-slider> -->
        <!-- 开关 -->
        <el-switch v-if="item.type === 'Switch'" v-model="searchData[item.prop]" :size="item.size"
          @change="item.change && item.change(searchData[item.prop])"></el-switch>
      </el-form-item>
      <el-form-item inline v-if="isHandle" class="formItem" v-for="(item, index) in searchHandle" :key="index">
        <el-button v-if="item.isPermitted ? item.isPermitted() : true" :type="item.type" :size="item.size || formSize" @click="item.handle()">{{ item.label }}</el-button>
      </el-form-item>
    </el-form>
    <!-- <el-form class="formT" :size="formSize" inline v-if="isHandle">
      <el-form-item
        class="formTItem"
        v-for="(item, index) in searchHandle"
        :key="index"
      >
        <el-button
          :type="item.type"
          :size="item.size || formSize"
          @click="item.handle()"
          >{{ item.label }}</el-button
        >
      </el-form-item>
    </el-form> -->
  </div>
</template>

<script>
export default {
  name: "SearchForm",
  props: {
    isHandle: {
      type: Boolean,
      default: true
    },
    labelWidth: {
      type: String,
      default: "50px"
    },
    formSize: {
      type: String,
      default: "mini"
    },
    searchForm: {
      type: Array,
      default: []
    },
    searchHandle: {
      type: Array,
      default: () => []
    },
    searchData: {
      type: Object,
      default: {}
    }
  },
  data() {
    return {};
  },
  methods: {}
};
</script>
<style lang="scss">
.ces-search {
  display: flex;
  // justify-content: space-between;

  align-items: center;

  .form {
    // flex-shrink: 0;
  }

  // .formT {
  //   flex-shrink: 0;
  // }
  .formItem {
    .el-form-item__label {
      width: 80px !important;
    }

    // formItem 间距
    margin-right: 12px;
  }

  // .formTItem {
  //   // formItem 间距
  //   margin-right: 12px;
  // }
}
</style>
