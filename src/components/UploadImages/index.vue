<template>
  <div style="margin-top:12px">
    <el-upload
      action=""
      :accept="accept"
      ref="uploadFile"
      list-type="picture-card"
      :before-upload="beforeAvatarUpload"
      :http-request="httpRequest"
      :data="aLiData"
      :on-preview="handlePictureCardPreview"
      :on-remove="handleRemove"
      :on-change="handleEditChange"
      :limit="limit"
      :file-list="fileList"
      :disabled="isDisabled"
      :class="{ hide: hideUploadBtn }"
    >
      <!-- :class="{ hide: hideUploadBtn }" -->
      <i class="el-icon-plus"></i>
    </el-upload>
    <el-dialog :visible.sync="dialogVisible" append-to-body>
      <img width="100%" :src="dialogImageUrl" alt="" />
    </el-dialog>
    <!---提示--->
    <slot name="tip">this is metion</slot>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "UploadImages",
  model: {
    prop: "value"
  },
  props: {
    //配合v-model 加载初始值
    value: {
      // type: [Array, String],
      required: true
    },
    limit: {
      type: Number,
      default: 1
    },
    alias: {
      type: String,
      default: ""
    },
    isDisabled: {
      type: Boolean,
      default: false
    },
    accept: {
      type: String,
      default: "image/jpeg,image/png"
    }
  },
  data() {
    return {
      dialogImageUrl: "",
      banner_list: [],
      dialogVisible: false,
      aLiData: {},
      fileList: [],
      hideUploadBtn: false
    };
  },

  methods: {
    // 上传前
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isPNG = file.type === "image/png";
      // const isLt2M = file.size / 1024 / 1024 < 2;
      if (!isJPG && !isPNG) {
        this.$message.error("图片只能是 JPG/PNG 格式!");
        return false;
      }
      // return this.getAliData(file);
    },
    //从后台获取第一步所需的数据
    async httpRequest(File) {
      try {
        let formData = new FormData();
        console.log("iamge alias: ", this.alias);
        formData.append("file", File.file);
        formData.append("alias", this.alias);

        const res = await axios({
          method: "post",
          url: "app/upload/file",
          data: formData,
          headers: {
            "Content-Type": "multipart/form-data"
          }
        });
        if (res.success) {
          this.aLiData = res.data;
          // 传值
          let obj = {
            url: this.aLiData.url
          };

          this.banner_list.push(obj);
          this.fileList = JSON.parse(JSON.stringify(this.banner_list));

          if (this.limit === 1) {
            this.$emit("input", this.banner_list[0].url);
          } else {
            this.$emit("input", this.banner_list.map(item => item.url));
          }
          return true;
        } else {
          const idx = this.$refs.uploadFile.uploadFiles.findIndex(
            item => item.uid === File.file.uid
          );
          this.$refs.uploadFile.uploadFiles.splice(idx, 1);
          this.hideUploadBtn = this.fileList.length >= this.limit;
          return false;
        }
      } catch (e) {
        return false;
      } finally {
      }
    },

    handleEditChange(file, fileList) {
      console.log("fileList ", fileList);
      this.hideUploadBtn = fileList.length >= this.limit;
    },
    handleRemove(file, fileList) {
      this.banner_list = JSON.parse(JSON.stringify(fileList));
      this.fileList = fileList;
      if (this.limit === 1) {
        this.$emit("input", "");
      } else {
        this.$emit("input", this.fileList.map(item => item.url));
      }

      this.hideUploadBtn = fileList.length >= this.limit;
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    }
  },
  watch: {
    //赋值
    value: {
      handler: function(newValue, oldValue) {
        if (this.limit === 1) {
          //newValue 类型字符串 or undefined
          if (typeof newValue == "undefined") {
            this.banner_list = [];
          } else if (typeof newValue == "string") {
            this.banner_list = !newValue ? [] : [{ url: newValue }];
          }
        } else {
          if (typeof newValue == "undefined") {
            this.banner_list = [];
          } else if (typeof newValue == "string") {
            this.banner_list = !newValue ? [] : [{ url: newValue }];
          } else {
            //newValue 类型数组 or undefined
            this.banner_list = !newValue
              ? []
              : newValue.map(url => {
                  return { url: url };
                });
          }
        }
        this.fileList = JSON.parse(JSON.stringify(this.banner_list));
        this.hideUploadBtn = this.fileList.length >= this.limit;
      },
      immediate: true
    }
  }
};
</script>
<style lang="scss">
.hide {
  .el-upload--picture-card {
    display: none !important;
  }
}
.el-upload-list__item {
  transition: none !important;
}
</style>
