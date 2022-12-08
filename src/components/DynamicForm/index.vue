<template>
  <div class="dynamic-form">
    <el-form
      label-position="top"
      :model="dynamicForm"
      ref="ruleForm"
      label-width="100px"
      class="demo-ruleForm"
    >
      <!--  :rules="dynamicFormrules" -->
      <el-form-item
        v-for="item in dataList"
        :label="item.name"
        :key="item.id"
        :prop="item.id"
        :rules="dynamicFormrules[item.id]"
      >
        <!-- 输入框 -->
        <el-input
          clearable
          v-if="item.type == 1"
          v-model="dynamicForm[item.id]"
          :maxlength="item.lengthMax"
          :show-word-limit="item.lengthMax == 0 ? false : true"
          :size="size"
          :placeholder="`请输入${item.name}...`"
        >
        </el-input>
        <!-- 下拉 -->
        <!-- 此处添加特例被访人远程搜索
          :filterable="item.alias == 'employeeId'"
          :remote="item.alias == 'employeeId'"
          :remote-method="remoteMethod"
          :loading="searchloading"
         -->
        <el-select
          clearable
          v-if="item.type === 2"
          v-model="dynamicForm[item.id]"
          :multiple="item.isArray"
          :size="size"
          style="width:100%"
          :placeholder="selcetPlaceholder(item)"
          :filterable="item.isFilterable"
          :remote="item.isRemote"
          :remote-method="item.remoteMethod"
          @change="selectChange(dynamicForm[item.id], item)"
        >
          <el-option
            v-for="op in item.data"
            :key="op.value"
            :label="op.label"
            :value="op.value"
          ></el-option>
        </el-select>
        <!-- 文本域 多行输入 -->
        <el-input
          clearable
          v-if="item.type == 3"
          v-model="dynamicForm[item.id]"
          :size="size"
          type="textarea"
          :maxlength="item.lengthMax"
          :placeholder="`请输入${item.name}...`"
          :show-word-limit="item.lengthMax == 0 ? false : true"
        >
        </el-input>
        <!-- 日期时间 -->
        <el-date-picker
          v-if="item.type == 4"
          v-model="dynamicForm[item.id]"
          :type="item.isRange ? 'datetimerange' : 'datetime'"
          prefix-icon="el-icon-date"
          :start-placeholder="`开始${item.name}...`"
          :end-placeholder="`截止${item.name}...`"
          :placeholder="`选择${item.name}...`"
          :size="size"
          range-separator="~"
          value-format="yyyy-MM-dd HH:mm:ss"
          style="width:100%;"
          :picker-options="item.pickerOptions"
        >
        </el-date-picker>
        <!-- 图片 -->
        <UploadImages
          v-if="item.type == 5"
          v-model="dynamicForm[item.id]"
          :limit="item.itemMax"
          :alias="item.alias"
        >
          <template>
            <div
              class="el-upload__tip"
              style="color: #E6A23C;line-height: 26px;margin: 0"
              slot="tip"
            >
              只能上传jpg/png文件，且不超过2MB，建议尺寸大小为328px*328px的图片
            </div>
          </template>
        </UploadImages>
        <!-- 数字 目前没有做最大值限制 -->
        <!-- <el-input-number
          v-if="item.type == 6"
          v-model="dynamicForm[item.id]"
          style="width:50%"
          :size="size"
          :min="0"
          :placeholder="`请输入${item.name}...`"
        ></el-input-number> -->
        <el-input
          clearable
          v-if="item.type == 6"
          v-model.number="dynamicForm[item.id]"
          :maxlength="item.lengthMax"
          :show-word-limit="item.lengthMax == 0 ? false : true"
          :size="size"
          :placeholder="`请输入${item.name}...`"
        >
        </el-input>
        <!-- 日期 -->
        <el-date-picker
          v-if="item.type == 7"
          v-model="dynamicForm[item.id]"
          :type="item.isRange ? 'daterange' : 'date'"
          prefix-icon="el-icon-date"
          :start-placeholder="`开始${item.name}...`"
          :end-placeholder="`截止${item.name}...`"
          :placeholder="`请选择${item.name}...`"
          :size="size"
          range-separator="~"
          style="width:100%;"
          value-format="yyyy-MM-dd"
        >
        </el-date-picker>
        <!-- 树形选择器 -->
        <el-select
          v-if="item.type == 8"
          size="small"
          v-model="valueData"
          :placeholder="`请选择${item.name}`"
          :popper-append-to-body="false"
          style="width:100%;"
        >
          <el-option value="[]" class="setstyle" disabled>
            <el-tree
              :data="item.data"
              :props="defaultProps"
              ref="tree"
              :highlight-current="true"
              @node-click="data => handleNodeClick(data, item.id)"
              :expand-on-click-node="false"
            ></el-tree>
          </el-option>
        </el-select>
        <!-- 地域选择器 -->
        <el-cascader
          v-if="item.type == 9"
          size="small"
          style="width:100%"
          :options="regionOptions"
          v-model="dynamicForm[item.id]"
          :placeholder="`请输入${item.name}...`"
          clearable
        >
        </el-cascader>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
// 级联
import { regionData, CodeToText, TextToCode } from "element-china-area-data";
import UploadImages from "@/components/UploadImages";

import moment from "moment";
export default {
  name: "DynamicForm",
  components: {
    UploadImages
  },
  props: {
    dynamicFormData: {
      type: Array,
      default: []
    },
    departmentId: {
      type: String,
      default: ""
    },
    // 判断部门类型
    departmentType: {
      type: Number,
      default: 0
    },
    size: {
      type: String,
      default: "small"
    },
    detailForm: {
      type: Object,
      default() {
        return {};
      }
    }
  },
  data() {
    // var checkValueData = (rule, value, callback) => {
    //   if (value === "") {
    //     callback(new Error("请输入部门名称"));
    //   } else {
    //     callback();
    //   }
    // };
    return {
      regionOptions: regionData,
      dynamicForm: {},
      dynamicFormrules: {},
      dataList: [],
      valueData: "",
      defaultProps: {
        children: "child",
        label: "label"
      },
      departmentTreeList: [],
      searchloading: false
    };
  },
  methods: {
    selectChange(type, item) {
      // 内写的时候 门禁到期时间需要做出对应更改
      if (this.departmentType == 2 && item.alias == "type") {
        let foundItem = this.dataList.find(
          item => item.alias == "entranceAccessPermissionEndTime"
        );
        if (foundItem.meta) {
          this.$set(
            this.dynamicForm,
            foundItem.id,
            foundItem.meta[`max[type=${type}]`]
          );

          foundItem.pickerOptions = {
            disabledDate: time => {
              var timestamp = Date.parse(
                new Date(foundItem.meta[`max[type=${type}]`])
              );
              return timestamp < time.getTime();
            }
          };
        }
      }
      //内协下 根据人员类型 判断体检时间必填否
      if (this.departmentType == 2 && item.alias == "type") {
        this.dataList.forEach(item_2 => {
          if (item_2.alias == "healthCheckTime") {
            this.$set(this.dynamicFormrules, item_2.id, [
              {
                required: type == 1 ? true : false,
                message: `请选择${item_2.name}`,
                trigger: "change"
              }
            ]);
          }
        });
      }
    },
    // 验证
    verify(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          // 抛出合法表单值
          this.$emit("submitForm", this.dynamicForm);
        } else {
          this.$message.error("信息填写有误！");
          console.log("error submit!!");
          // this.$emit("submitForm", {});
          return false;
        }
      });
    },
    // 充值
    resetFields(formName) {
      this.valueData = "";
      this.$refs[formName].resetFields();
    },
    clearValidate() {
      this.$refs["ruleForm"].clearValidate();
    },
    selcetPlaceholder(item) {
      // 被访人
      if (item.alias == "employeeId") {
        return `请搜索${item.name}...`;
      } else {
        return `请选择${item.name}...`;
      }
    },
    //赋值
    handleNodeClick(data, item) {
      console.log("data", data);
      console.log("item", item);
      this.valueData = data.label; //展示部分
      // this.dynamicForm[item] = data.value; //传参
      this.$set(this.dynamicForm, item, data.value);
    },
    // // fillForm
    // fillForm(row, form) {
    //   //   this.$refs["ruleForm"].clearValidate();
    //   this.$nextTick(() => {
    //     for (const key in form) {
    //       form[key] = row[key];
    //     }
    //   });
    // },
    getTreeName(id, data) {
      for (let i = 0; i < data.length; i++) {
        let a = data[i];
        if (a.value == id) {
          return a.label;
        } else {
          if (a.child && a.child.length > 0) {
            let res = this.getTreeName(id, a.child);
            if (res) {
              return res;
            }
          }
        }
      }
    }
  },
  mounted() {
    console.log("time", moment().format("YYYY-MM-DD hh:mm:ss"));
  },
  watch: {
    dynamicFormData: {
      handler(newVal, oldVal) {
        this.dataList = newVal;
        console.log("newVal", newVal);
        //添加rules
        newVal.forEach(item => {
          if (item.type == 2 || item.type == 5) {
            if (item.regular) {
              this.dynamicFormrules[item.id] = [
                {
                  required: item.isRequired,
                  message: `请选择${item.name}`,
                  trigger: "change"
                },
                {
                  pattern: new RegExp(item.regular),
                  message: "不符合格式",
                  trigger: "change"
                }
              ];
            } else {
              this.dynamicFormrules[item.id] = [
                {
                  required: item.isRequired,
                  message: `请选择${item.name}`,
                  trigger: "change"
                }
              ];
            }
          } else {
            if (item.regular) {
              this.dynamicFormrules[item.id] = [
                {
                  required: item.isRequired,
                  message: `请输入${item.name}`,
                  trigger: "blur"
                },
                {
                  pattern: new RegExp(item.regular),
                  message: "不符合格式",
                  trigger: "blur"
                }
              ];
            } else {
              this.dynamicFormrules[item.id] = [
                {
                  required: item.isRequired,
                  message: `请输入${item.name}`,
                  trigger: "blur"
                }
              ];
            }
          }
          // 如果是判断为部门 启用自定义校验 自定义
          if (item.alias == "departmentId") {
            this.dynamicFormrules[item.id] = [
              {
                validator: (rule, value, callback) => {
                  if (!value) {
                    callback(new Error("请输入部门名称"));
                  } else {
                    callback();
                  }
                },
                required: item.isRequired,
                trigger: "change"
              }
            ];
          }

          // 身份证号格式校验
          if (item.alias == "idCard") {
            this.dynamicFormrules[item.id] = [
              {
                required: item.isRequired,
                message: `请输入${item.name}`,
                trigger: "blur"
              },
              {
                pattern: /^[1-9]\d{5}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9xX]$/,
                message: "不符合身份证格式",
                trigger: "blur"
              }
            ];
          }
          // 被访人
          if (item.alias == "employeeId") {
            this.dynamicFormrules[item.id] = [
              {
                required: item.isRequired,
                message: `请搜索${item.name}`,
                trigger: "change"
              }
            ];
          }

          //内协下 根据人员类型 判断体检时间必填否
          if (this.departmentType == 2 && item.alias == "healthCheckTime") {
            newVal.forEach(item_2 => {
              if (item_2.alias == "type") {
                this.dynamicFormrules[item.id] = [
                  {
                    required: item_2.value.data == 1 ? true : false,
                    message: `请选择${item.name}`,
                    trigger: "change"
                  }
                ];
              }
            });
          }
        });
      },
      immediate: false
    },
    detailForm: {
      handler(newVal, oldVal) {
        //回显！ detail
        this.dynamicForm = newVal;
        if (this.departmentId) {
          // 获取部门树
          let departmentTree = this.dynamicFormData.find(
            item => item.id == this.departmentId
          );
          this.valueData = this.getTreeName(
            this.dynamicForm[this.departmentId],
            departmentTree.data
          );
          // console.log("this.valueData",this.valueData);
        }
      },
      immediate: false
    }
  }
};
</script>
<style lang="scss">
.setstyle {
  height: auto;
  padding: 0 !important;
  margin: 0;
  overflow: auto;
  cursor: default !important;
}
</style>
