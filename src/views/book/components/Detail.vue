<template>
  <el-form ref="postForm" :model="postForm" :rules="rules">
    <sticky :class-name="'sub-navbar '">
      <el-button v-if="!isEdit" @click="showGuide">显示帮助</el-button>
      <el-button
        v-loading="loading"
        @click="submitForm"
        type="success"
        style="margin-left: 10px"
      >
        {{ isEdit ? "编辑电子书" : "新增电子书" }}
      </el-button>
    </sticky>
    <div class="detail-container">
      <el-row>
        <Warning />
        <el-col :span="24">
          <ebook-upload
            :file-list="fileList"
            :disabled="isEdit"
            @onSuccess="onUploadSuccess"
            @onRemove="onUploadRemove"
          />
        </el-col>
        <el-col :span="24">
          <el-form-item prop="title">
            <MDinput
              v-model="postForm.title"
              :maxlength="100"
              name="name"
              required
              >书名</MDinput
            >
          </el-form-item>
          <el-row>
            <el-col :span="12">
              <el-form-item
                prop="author"
                label="作者"
                :label-width="labelWidth"
              >
                <el-input v-model="postForm.author" placeholder="作者" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item
                prop="publisher"
                label="出版社"
                :label-width="labelWidth"
              >
                <el-input v-model="postForm.publisher" placeholder="出版社" />
              </el-form-item>
            </el-col>
          </el-row>

          <el-row>
            <el-col :span="12">
              <el-form-item
                prop="language"
                label="语言"
                :label-width="labelWidth"
              >
                <el-input v-model="postForm.language" placeholder="语言" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="根文件" :label-width="labelWidth">
                <el-input
                  prop="rootFile"
                  v-model="postForm.rootFile"
                  placeholder="根文件"
                  disabled
                />
              </el-form-item>
            </el-col>
          </el-row>

          <el-row>
            <el-col :span="12">
              <el-form-item
                prop="filePath"
                label="文件路径"
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.filePath"
                  placeholder="文件路径"
                  disabled
                />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item
                prop="unzipPath"
                label="解压路径"
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.unzipPath"
                  placeholder="解压路径"
                  disabled
                />
              </el-form-item>
            </el-col>
          </el-row>

          <el-row>
            <el-col :span="12">
              <el-form-item
                prop="coverPath"
                label="封面路径"
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.coverPath"
                  placeholder="封面路径"
                  disabled
                />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item
                prop="originalName"
                label="文件名称"
                :label-width="labelWidth"
              >
                <el-input
                  v-model="postForm.originalName"
                  placeholder="文件名称"
                  disabled
                />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <el-form-item prop="cover" label="封面" :label-width="labelWidth">
                <a v-if="postForm.cover" :href="postForm.cover" target="_blank">
                  <img :src="postForm.cover" class="preview-img" />
                </a>
                <span v-else>无</span>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="24">
              <el-form-item label="目录" :label-width="labelWidth">
                <div
                  v-if="contentTree && contentTree.length > 0"
                  class="contents-wrapper"
                >
                  <el-tree
                    :data="contentTree"
                    @node-click="onContentClick"
                  ></el-tree>
                </div>
                <span v-else>无</span>
              </el-form-item>
            </el-col>
          </el-row>
        </el-col>
      </el-row>
    </div>
  </el-form>
</template>

<script>
import Sticky from "../../../components/Sticky/index";
import Warning from "./Warning";
import EbookUpload from "../../../components/EbookUpload";
import MDinput from "../../../components/MDinput";
import { createBook, getBook, updateBook } from "../../../api/book";
// const defaultForm = {
//   title: "",
//   author: "",
//   publisher: "",
//   language: "",
//   rootFile: "",
//   cover: "",
//   url: "",
//   originalName: "",
//   fileName: "",
//   coverPath: "",
//   filePath: "",
//   unzipPath: "",
// };
const fields = {
  title: "标题",
  author: "作者",
  publisher: "出版社",
  language: "语言",
};

export default {
  name: "Detail",
  components: {
    Sticky,
    Warning,
    EbookUpload,
    MDinput,
  },
  data() {
    const validateRequire = (rule, value, callback) => {
      // console.log(rule, value);
      if (value.length === 0) {
        callback(new Error(fields[rule.field] + "为必须填写"));
      } else {
        callback();
      }
    };
    return {
      loading: false,
      postForm: {},
      fileList: [],
      labelWidth: "120px",
      contentTree: [],
      rules: {
        title: [{ validator: validateRequire }],
        author: [{ validator: validateRequire }],
        publisher: [{ validator: validateRequire }],
        language: [{ validator: validateRequire }],
      },
    };
  },
  props: {
    isEdit: Boolean,
  },
  methods: {
    onContentClick(data) {
      console.log(data);
      if (data.text) {
        window.open(data.text);
      }
    },
    showGuide() {
      console.log("showguide");
    },
    submitForm() {
      const  onSuccess = (response)=> {
        const { msg } = response;
        this.$notify({
          title: "操作成功",
          message: msg,
          type: "success",
          duration: 2000,
        });
        this.loading = false;
      }
      if (!this.loading) {
        this.loading = true;
        this.$refs.postForm.validate((valid, fields) => {
          console.log(valid, fields);
          if (valid) {
            const book = Object.assign({}, this.postForm);
            delete book.contentTree;
            if (!this.isEdit) {
              createBook(book)
                .then((response) => {
                  onSuccess(response);
                  this.setDefault();
                })
                .catch(() => {
                  this.loading = false;
                });
            } else {
              updateBook(book)
                .then((response) => {
                  onSuccess(response);
                })
                .catch(() => {
                  this.loading = false;
                });
            }
          } else {
            const message = fields[Object.keys(fields)[0]][0].message;
            this.$message({ message, type: "error" });
            this.loading = false;
          }
        });
      }
    },
    setData(data) {
      const {
        title,
        author,
        publisher,
        language,
        rootFile,
        cover,
        url,
        originalName,
        contents,
        fileName,
        coverPath,
        filePath,
        unzipPath,
        chapterTree,
      } = data;
      this.contentTree = contents;
      this.postForm = {
        ...this.postForm,
        title,
        author,
        publisher,
        language,
        rootFile,
        cover,
        url,
        originalName,
        contents,
        fileName,
        coverPath,
        filePath,
        unzipPath,
      };
      this.fileList = [{ name: originalName || fileName, url }];
    },
    setDefault() {
      // this.postForm = Object.assign({}, defaultForm);
      this.chapterTree = [];
      this.fileList = [];
      this.$refs.postForm.resetFields();
    },
    onUploadSuccess(data) {
      console.log("onUploadSuccess", data);
      this.setData(data);
    },
    onUploadRemove() {
      this.setDefault();
    },
    getBookData(fileName) {
      getBook(fileName).then((response) => {
        this.setData(response.data);
      });
    },
  },
  created() {
    console.log(this.$route.params);
    if (this.isEdit) {
      const fileName = this.$route.params.fileName;
      this.getBookData(fileName);
    }
  },
};
</script>

<style lang="scss">
.detail-container {
  padding: 40px 50px 20px;
  .preview-img {
    width: 200px;
    height: 270px;
  }
}
</style>
