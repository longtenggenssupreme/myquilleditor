<template>
  <div id="app">
    <!-- input 上传控件 -->
    <input type="file" name="file" multiple ref="myinpufile" class="myinpufile" id="myinpufile">
    <button @click="buttonclick">{{inputfilename}}</button>

    <!-- el-upload 封装的上传控件 -->
    <div>编辑器的值：{{content}}</div>
    <div class="myshuoming">封装elment-ui的上传upload控件，重写上传逻辑</div>
    <el-upload class="upload-demo" drag multiple :file-list="fileList" action="http://localhost:8088/test"
      :on-remove="handleRemove" :before-remove="beforeRemove" :limit="3" :http-request="httpupload"
      :on-exceed="handleExceed" :before-upload="beforeUpload" list-type="picture" :name="uploadname">
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
      <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
    </el-upload>

    <!-- tinymce富文本编辑器tinymceeditor -->
    <!-- <tinymceeditor v-model="content" ref="tinymceeditor"></tinymceeditor> -->
    
    <div class="myshuoming">vue-quill-editor富文本编辑器quilleditor</div>
    <!-- vue-quill-editor富文本编辑器quilleditor -->
    <!-- <quill-editor v-model="content" ref="QuillEditor"></quill-editor> -->
    <!-- 自定义工具栏 -->
    <quill-editor v-model="content" ref="QuillEditor" :options="editorOption"></quill-editor>
    
    <!-- quill 富文本编辑器 与vue-quill-editor不同 -->
    <div class="myshuoming">quill 富文本编辑器 与vue-quill-editor不同</div>    
    <!-- <TestQuillEditor v-model="content" ref="myQuillEditor"></TestQuillEditor> -->

    <!-- <myqull v-model="content" ref="myQuillEditor"></myqull> -->
    <myquill :content.sync="content" ref="myQuillEditor"></myquill>
     
  </div>
</template>

<script>

// tinymce富文本编辑器tinymceeditor
// import tinymceeditor from './components/myeditor.vue'

//vue-quill-editor富文本编辑器quilleditor
import { quillEditor } from "vue-quill-editor";
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";
import "quill/dist/quill.bubble.css";

import TestQuillEditor from './components/tes.vue';

import myqull from './components/myqulldrag.vue';
import myquill from './components/myquill.vue';



// 工具栏配置
const toolbarOptions = [
  ["bold", "italic", "underline", "strike"], // toggled buttons
  ["blockquote", "code-block"],

  [{ header: 1 }, { header: 2 }], // custom button values
  [{ list: "ordered" }, { list: "bullet" }],
  [{ script: "sub" }, { script: "super" }], // superscript/subscript
  [{ indent: "-1" }, { indent: "+1" }], // outdent/indent
  [{ direction: "rtl" }], // text direction

  [{ size: ["small", false, "large", "huge"] }], // custom dropdown
  [{ header: [1, 2, 3, 4, 5, 6, false] }],

  [{ color: [] }, { background: [] }], // dropdown with defaults from theme
  [{ font: [] }],
  [{ align: [] }],
  ["link", "image", "video"],
  ["clean"] // remove formatting button
];
export default {
  name: "App",
  components: {
    // tinymceeditor,//tinymce富文本编辑器tinymceeditor

    quillEditor,//vue-quill-editor富文本编辑器quilleditor

    // myqull,
    myquill,
    TestQuillEditor
  },
  data() {
    return {
      inputfilename:"上传选择文件",
      uploadname: "",
      // fileList: [],
      fileList: [
        // {
        //   name: "food.jpeg",
        //   url:"https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100"
        // },
        // {
        //   name: "food2.jpeg",
        //   url:"https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100"
        // }
      ],
      content: "quill",
      editorOption: {
        modules: {
          toolbar: {
            container: toolbarOptions, // 工具栏
            // container: ['bold', 'italic', 'underline', 'strike', 'blockquote', 'code-block', {'header': 1}, {'header': 2},{'list': 'ordered'}, {'list': 'bullet'},{'indent': '-1'}, {'indent': '+1'},{'direction': 'rtl'},{'size': ['small', false, 'large', 'huge']}],
            handlers: {
              image: function(value) {
                if (value) {
                  console.log(value);
                  alert("自定义图片");
                } else {
                  // this.quill.format('image', false);
                  QuillEditor.format("image", false);
                }
              }
            }
          }
        }
      }
    };
  },
  methods: {
    //使用input file上传文件,点击上传选择的文件
    buttonclick(){
      var but1= document.getElementById("myinpufile").files;
      console.log(but1);
       var but4= this.$refs.myinpufile.files;
       console.log(but4);
       this.inputfilename='ss';
    },
    //el-upload文件上传的处理逻辑
    beforeUpload(file) {
      console.log(file);
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },
    httpupload(option) {
      // alert("开始上传");
      if (typeof XMLHttpRequest === "undefined") {
        return;
      }

      var xhr = new XMLHttpRequest();
      var action = option.action;

      if (xhr.upload) {
        xhr.upload.onprogress = function progress(e) {
          if (e.total > 0) {
            e.percent = (e.loaded / e.total) * 100;
          }
          option.onProgress(e);
        };
      }

      var formData = new FormData();

      if (option.data) {
        Object.keys(option.data).forEach(function(key) {
          formData.append(key, option.data[key]);
        });
      }
      // formData.append(option.filename, option.file, option.file.name);
      //添加文件
      // formData.append("files", option.file, option.file.name);

      //上传图片大于1M进行压缩
      if (option.file.size / 1024 > 1025) {
        this.photoCompress(option.file, { quality: 0.2 }, function(
          base64Codes
        ) {
          var bl = convertBase64UrlToBlob(base64Codes);
          formData.append("files", bl, option.file.name); // 文件对象
          //上传
          // $.ajax({});
        });
      } else {
        formData.append("files", option.file, option.file.name);
        //上传
        // $.ajax({});
      }

      xhr.onerror = function error(e) {
        option.onError(e);
      };

      var text = "成功;";
      xhr.onload = function onload() {
        if (xhr.status < 200 || xhr.status >= 300) {
          // return option.onError(this.getError(action, option, xhr));
          text = xhr.responseText || xhr.response;
          alert("失败" + text);
        }
        // alert(text);
        // this.handlSuccess(xhr.res, option.file);
        if (option.file) {
          option.file.status = "success";
          option.file.response = xhr.res;

          option.onSuccess(xhr.res, option.file, option.uploadFiles);
          // option.onChange(option.file, option.uploadFiles);
        }
      };

      xhr.open("post", action, true);

      // if (option.withCredentials && "withCredentials" in xhr) {
      //   xhr.withCredentials = true;
      // }

      // var headers = option.headers || {};

      // for (var item in headers) {
      //   if (headers.hasOwnProperty(item) && headers[item] !== null) {
      //     xhr.setRequestHeader(item, headers[item]);
      //   }
      // }
      xhr.send(formData);
      return xhr;
    },
    uploadtest() {
      if (typeof XMLHttpRequest === "undefined") {
        return;
      }
      var xmlhttp = new XMLHttpRequest();
      xmlhttp.onreadystatechange = function() {
        if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
          alert(xmlhttp.responseText || xmlhttp.responseText);
        }
      };
      var formData = new FormData();

      formData.append("haha", "123");
      // xmlhttp.open("post", "https://jsonplaceholder.typicode.com/posts/", true);
      xmlhttp.open("post", "http://localhost:8084/test", true);
      xmlhttp.send(formData);
    },
    handleRemove(file, fileList) {
      // alert("handleRemove");
      console.log(file, fileList);
    },
    handlePreview(file) {
      alert("handlePreview");
      console.log(file);
    },
    handleExceed(files, fileList) {
      this.$message.warning(
        `当前限制选择 3 个文件，本次选择了 ${
          files.length
        } 个文件，共选择了 ${files.length + fileList.length} 个文件`
      );
    },
    beforeRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`);
    },
    handlSuccess(res, file) {
      var file = this.getFile(rawFile);

      if (file) {
        file.status = "success";
        file.response = res;

        this.onSuccess(res, file, this.uploadFiles);
        this.onChange(file, this.uploadFiles);
      }
      console.log(res);
      this.imageUrl = URL.createObjectURL(file.raw);
    },
    photoCompress(file, objCompressed, objDiv) {
      var ready = new FileReader();
      ready.readAsDataURL(file);
      ready.onload = function() {
        var fileResult = this.result;
        canvasDataURL(fileResult, objCompressed, objDiv);
      };
    },
    canvasDataURL(path, objCompressed, callback) {
      var img = new Image();
      img.src = path;
      img.onload = function() {
        var that = this;
        //默认压缩后图片规格
        var quality = 0.5;
        var w = that.width;
        var h = that.height;
        var scale = w / h;
        //实际要求
        w = objCompressed.width || w;
        h = objCompressed.height || w / scale;
        if (
          objCompressed.quality &&
          objCompressed.quality > 0 &&
          objCompressed.quality <= 1
        ) {
          quality = objCompressed.quality;
        }

        //生成canvas
        var canvas = document.createElement("canvas");
        var ctx = canvas.getContext("2d");
        // 创建属性节点
        var anw = document.createAttribute("width");
        anw.nodeValue = w;
        var anh = document.createAttribute("height");
        anh.nodeValue = h;
        canvas.setAttributeNode(anw);
        canvas.setAttributeNode(anh);
        ctx.drawImage(that, 0, 0, w, h);

        var base64 = canvas.toDataURL("image/jpeg", quality);
        // 回调函数返回base64的值
        callback(base64);
      };
    },
    //上传文件压缩的处理
    convertBase64UrlToBlob(urlData) {
      var arr = urlData.split(","),
        mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]),
        n = bstr.length,
        u8arr = new Uint8Array(n);
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }
      return new Blob([u8arr], { type: mime });
    }
  }
};
</script>
<style scoped>
.myinpufile {
  margin-bottom: 10px;
}
.myshuoming {
  margin-bottom: 10px;
  margin-top: 10px;
  color: rgb(102, 185, 102);
}
.upload-demo {
  margin-bottom: 20px;
  margin-top: 10px;
}


.ql-container.ql-snow {
  margin-bottom: 22px;
}
.ql-editor {
  height: 500px;
}
</style>
