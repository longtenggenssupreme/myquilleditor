<template>
  <div>
    <!-- <div id="toolbar">
      <span class="ql-formats">
        <button class="ql-image">image123</button>
        <button class="ql-video">video</button>
      </span>
    </div> -->
    <div id="editor">
    </div>
    <!-- 测试video标签 -->
    <!-- <video src="https://www.w3school.com.cn/i/movie.mp4" controls="controls" width="100%" height="100%" webkit-playsinline="true" playsinline="true" x5-playsinline="true"></video> -->
    <input id="uploadImg" ref="uploadImg" type="file" style="display:none" accept="image/png, image/jpeg, image/gif"
      @change="uploadImage">
    <input id="uploadVideo" ref="uploadVideo" type="file" style="display:none" accept="video/*" @change="uploadVideo">
  </div>
</template>
 
<script>
import Quill from "quill";
const BlockEmbed = Quill.import("blots/block/embed");
class VideoBlot extends BlockEmbed {
  static create(value) {
    let node = super.create();
    node.setAttribute("src", value.url);
    node.setAttribute("controls", value.controls);
    node.setAttribute("width", value.width);
    node.setAttribute("height", value.height);
    node.setAttribute("webkit-playsinline", true);
    node.setAttribute("playsinline", true);
    node.setAttribute("x5-playsinline", true);
    return node;
  }

  static value(node) {
    return {
      url: node.getAttribute("src"),
      controls: node.getAttribute("controls"),
      width: node.getAttribute("width"),
      height: node.getAttribute("height")
    };
  }
}
VideoBlot.blotName = "simpleVideo";
VideoBlot.tagName = "video";
Quill.register(VideoBlot);
var toolbarOptions = [
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
  ["link", "image", "video", "formula"],
  ["clean"] // remove formatting button
];
export default {
  name: "myquill",
  props: {
    content: {
      type: String,
      default: "编辑器默认值"
    }
  },
  data() {
    return {
      contentchange: ""
    };
  },
  watch: {
    contentchange(newvalue, oldvalue) {
      this.$emit("update:content", newvalue);
    }
  },
  mounted() {
    this.initQuill();
  },
  beforeDestroy() {
    this.quill = null;
    delete this.quill;
  },
  methods: {
    initQuill() {
      const quill = new Quill("#editor", {
        theme: "snow",
        modules: {
          //   toolbar: '#toolbar'
          toolbar: {
            container: toolbarOptions // 工具栏
            //container: ['bold', 'italic', 'underline', 'strike', 'blockquote', 'code-block', {'header': 1}, {'header': 2},{'list': 'ordered'}, {'list': 'bullet'},{'indent': '-1'}, {'indent': '+1'},{'direction': 'rtl'},{'size': ['small', false, 'large', 'huge']}],
            //   handlers: {
            //       image: function(value) {
            //           if (value) {
            //               alert(value);
            //               alert("自定义图片");
            //               const input = document.querySelector('#uploadImg');
            //               input.value = '';
            //               input.click();
            //               } else {
            //                   this.quill.format('image', false);
            //                   editor.format("image", false);
            //               }
            //         }
            //     }
          }
        },
        // debug: 'info',
        placeholder: "我的编辑器占位符..."
        // readOnly: true,
      });
      this.quill = quill;
      this.quill
        .getModule("toolbar")
        .addHandler("image", this.uploadImageHandler);
      this.quill
        .getModule("toolbar")
        .addHandler("video", this.uploadVideoHandler);
      this.quill.setContents(this.content);
      this.quill.root.innerHTML = this.content; //初始化赋值
      //   this.quill.updateContents(new Delta().retain(6).delete(5).insert('Quill啊哈哈哈').retain(1, { bold: true }));
      const that = this;
      this.quill.on("editor-change", function(eventName, ...args) {
        if (that.quill.root) {
          that.contentchange = that.quill.root.innerHTML;
        }
      });
    },
    uploadImageSucess(type) {
      const addImageRange = this.quill.getSelection();
      const newRange = 0 + (addImageRange !== null ? addImageRange.index : 0);
      if (type === 1) {
        const url =
          "https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100";
        this.quill.insertEmbed(newRange, "image", url);
        this.quill.setSelection(1 + newRange);
      } else {
        const url = "https://www.w3school.com.cn/i/movie.mp4";
        this.quill.insertEmbed(newRange, "simpleVideo", {
          url,
          controls: "controls",
          width: "320",
          height: "240",
          autoplay: "autoplay"
        });
        this.quill.setSelection(1 + newRange);
      }
      //获取内容
      console.log(this.quill.getContents());
      console.log(this.quill.getText());
      var html = this.quill.root.innerHTML;
      // alert(html);
      console.log(html);
      var xx = this.quillGetHTML(this.quill.getContents());
      //  alert(xx);
      console.log(xx);
    },
    quillGetHTML(inputDelta) {
      //护球编辑器的内容
      var tempCont = document.createElement("div");
      new Quill(tempCont).setContents(inputDelta);
      return tempCont.getElementsByClassName("ql-editor")[0].innerHTML;
    },
    uploadImageHandler() {
      const input = document.querySelector("#uploadImg");
      input.value = "";
      input.click();
      // this.$refs.uploadImage.value = "";
      // this.$refs.uploadImage.click();
    },
    uploadImage(event) {
      if (typeof XMLHttpRequest === "undefined") {
        return;
      }
      var xhr = new XMLHttpRequest();
      if (xhr.upload) {
        xhr.upload.onprogress = function progress(e) {
          if (e.total > 0) {
            e.percent = (e.loaded / e.total) * 100;
          }
          //   option.onProgress(e);
        };
      }
      const formData = new FormData();
      //   formData.append('upload_file', event.target.files[0]);
      var file = event.target.files[0];
      const that = this;
      //上传图片大于1M进行压缩
      if (file.size / 1024 > 1025) {
        that.photoCompress(file, { quality: 0.2 }, function(base64Codes) {
          var bl = that.convertBase64UrlToBlob(base64Codes);
          formData.append("files", bl, file.name);
        });
      } else {
        formData.append("files", file, file.name);
      }
      xhr.onerror = function error(e) {
        alert("失败" + e);
      };
      var text = "成功";
      xhr.onload = function onload() {
        if (xhr.status < 200 || xhr.status >= 300) {
          text = xhr.responseText || xhr.response;
          alert("失败" + text);
        }
        console.log(text);
        //上传成功，根据xhr.response的返回信息中的url，将url插入到编辑区
        that.uploadImageSucess(1);
      };

      xhr.open("post", "http://localhost:8088/test", true);
      xhr.send(formData);
    },
    uploadVideoHandler() {
      const input = document.querySelector("#uploadVideo");
      input.value = "";
      input.click();
      // this.$refs.uploadVideo.value = "";
      // this.$refs.uploadVideo.click();
    },
    uploadVideo(event) {
      if (typeof XMLHttpRequest === "undefined") {
        return;
      }
      var xhr = new XMLHttpRequest();
      const formData = new FormData();
      formData.append("upload_file", event.target.files[0]);
      if (typeof XMLHttpRequest === "undefined") {
        return;
      }
      var xhr = new XMLHttpRequest();
      var text = "成功";
      const that = this;
      xhr.onload = function onload() {
        if (xhr.status < 200 || xhr.status >= 300) {
          // return option.onError(this.getError(action, option, xhr));
          text = xhr.responseText || xhr.response;
          alert("失败" + text);
        }
        console.log(text);
        that.uploadImageSucess(2);
      };

      xhr.open("post", "http://localhost:8088/test", true);
      xhr.send(formData);
    },
    photoCompress(file, objCompressed, objDiv) {
      var ready = new FileReader();
      ready.readAsDataURL(file);
      const that = this;
      ready.onload = function() {
        var fileResult = this.result;
        that.canvasDataURL(fileResult, objCompressed, objDiv);
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