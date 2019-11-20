<template>
  <div class="pb_rotation_edit" v-loading="loading">
    <div class="pb_title">
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/special' }">P&amp;B专题</el-breadcrumb-item>
        <el-breadcrumb-item>{{isadd?'添加':'编辑'}}</el-breadcrumb-item>
      </el-breadcrumb>
      <div class="button">
        <el-button type="primary" @click.native="save()" size="small" round>保存</el-button>
        <el-button size="small" @click.native="$router.push('/special')" round>取消</el-button>
      </div>
    </div>
    <div class="from pb_content">
      <div class="right_ck">
        <el-switch v-model="ck" active-text="是否启用"></el-switch>
      </div>
      <el-form label-width="80px">
        <el-form-item label="专题标题">
          <el-input v-model="v.title" placeholder="不超过20个字"></el-input>
        </el-form-item>
        <el-form-item label="专题描述">
          <el-input v-model="v.descriptions" placeholder="不超过20个字"></el-input>
        </el-form-item>
        <el-form-item label="专题作者">
          <el-input v-model="v.author" placeholder="不超过20个字"></el-input>
        </el-form-item>
        <el-form-item label="专题图片">
          <el-upload
            class="avatar-uploader"
            :on-success="successPreview"
            :before-upload="before_upload"
            :action="UpUrl"
            :data="postData"
            :show-file-list="false"
          >
            <img v-if="v.imagePath" :src="imgUrl + v.imagePath" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
           <p>请上传宽710px，高460px的图片</p>
        </el-form-item>
        <el-form-item label="专题正文">
          <div style="line-height:1;line-height: 1;height: 400px;display: grid;width: 600px;">
            <quill-editor
              v-model="v.content"
              ref="myQuillEditor"
              :options="editorOption"
              @blur="onEditorBlur($event)"
              @focus="onEditorFocus($event)"
              @change="onEditorChange($event)"
            ></quill-editor>
            <!-- 文件上传input 将它隐藏-->
            <el-upload
              class="upload-demo"
              :on-success="successPreview3"
              :before-upload="before_upload3"
              :action="UpUrl"
              :data="postData"
              :show-file-list="false"
              ref="upload"
              style="display:none"
            >
              <el-button size="small" type="primary" id="imgInput">点击上传</el-button>
            </el-upload>
          </div>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>
<script>
import { quillEditor } from "vue-quill-editor"; //调用编辑器
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";
import "quill/dist/quill.bubble.css";
export default {
  components: {
    quillEditor
  },
  data() {
    return {
      postData: {
        token: this.$parent.$parent.uptoken,
        key: this.$file_name()
      },
      ck: true,
      loading: false,
      isadd: true,
      v: {
        id: 0,
        title: "",
        imagePath: "",
        status: 0,
        content: "",
        author: "",
        descriptions: "",
        classifyid: 0,
        href: "",
        sort: 0
      },
      editorOption: {
        modules: {
          toolbar: [
            ["bold", "italic", "underline", "strike"],
            [{ color: [] }, { background: [] }],
            [{ align: [] }],
            ["image"],
            ["clean"]
          ]
        }
      },
      addRange: new Array()
    };
  },
  computed: {
    editor() {
      return this.$refs.myQuillEditor.quill;
    }
  },
  mounted() {
    this.$refs.myQuillEditor.quill
      .getModule("toolbar")
      .addHandler("image", this.imgHandler);
    if (parseInt(this.$route.query.id) > 0) {
      this.isadd = false;
      this.get_brandspecial();
    }
  },
  methods: {
    imgHandler(state) {
      this.addRange = this.$refs.myQuillEditor.quill.getSelection();
      if (state) {
        let fileInput = document.getElementById("imgInput");
        fileInput.click(); 
      }
    },
     before_upload3() {
      this.postData.key = "images/news/"+this.$file_name();
      this.loading = true;
    },
    successPreview3(response, file, fileList) {
      this.loading = false;
      let url = this.imgUrl + response.key;
      this.addRange = this.$refs.myQuillEditor.quill.getSelection();
      this.$refs.myQuillEditor.quill.insertEmbed(
        this.addRange !== null ? this.addRange.index : 0,
        "image",
        url
      ); // 调用编辑器的 insertEmbed 方法，插入URL
      this.$refs["upload"].clearFiles();
    },
    onEditorReady(editor) {
      // 准备编辑器
    },
    onEditorBlur() {}, // 失去焦点事件
    onEditorFocus() {}, // 获得焦点事件
    onEditorChange() {}, // 内容改变事件
    before_upload() {
      this.postData.key = this.$file_name();
    },
    successPreview(response, file, fileList) {
      this.v.imagePath = response.key;
    },
    get_brandspecial() {
      this.loading = true;
      this.$get("subject/view/" + this.$route.query.id)
        .then(res => {
          this.loading = false;
          this.v = res.data.entity;
          this.ck = this.v.status == 1;
        })
        .catch(() => {
          this.loading = false;
        });
    },
    save() {
      if (!this.v.title) {
        this.$message({
          type: "info",
          message: "请输入专题标题"
        });
        return;
      }
      if (!this.v.descriptions) {
        this.$message({
          type: "info",
          message: "请输入专题描述"
        });
        return;
      }
      if (!this.v.author) {
        this.$message({
          type: "info",
          message: "请输入专题作者"
        });
        return;
      }
      if (!this.v.imagePath) {
        this.$message({
          type: "info",
          message: "请添加专题图片"
        });
        return;
      }
      if (!this.v.content) {
        this.$message({
          type: "info",
          message: "请输入专题正文"
        });
        return;
      }
      this.v.status = 1;
      if (!this.ck) {
        this.v.status = 0;
      }
      if (!this.isadd) {
        delete this.v.page;
        delete this.v.rows;
        delete this.v.currentPage;
        delete this.v.createtime;
        delete this.v.sort;
        delete this.v.videopath;
      } else {
        delete this.v.id;
      }
      this.loading = true;
      this.$post("subject/save", this.v)
        .then(() => {
          this.loading = false;
          this.$message({
            type: "success",
            message: "保存成功"
          });
          if (this.isadd) {
            this.$router.push("/special");
          }
        })
        .catch(() => {
          this.loading = false;
        });
    }
  }
};
</script>