<template>
  <div>
    <div id="toolbar">
      <span class="ql-formats">
        <button class="ql-image">image</button>
        <button class="ql-video">video</button>
      </span>
    </div>
    <div id="editor">
      <p>Hello World!</p>
      <p>Some initial <strong>bold</strong> text</p>
      <p><br></p>
    </div>
    <!-- 测试video标签 -->
    <!-- <video src="https://www.w3school.com.cn/i/movie.mp4" controls="controls" width="100%" height="100%" webkit-playsinline="true" playsinline="true" x5-playsinline="true"></video> -->
    <input id="uploadImg" type="file"  style="display:none"  accept="image/png, image/jpeg, image/gif" @change="uploadImage">
    <input id="uploadVideo" type="file" style="display:none" accept="video/*" @change="uploadVideo">
  </div>
</template>
 
<script>
import Quill from 'quill'
const BlockEmbed = Quill.import('blots/block/embed')
class VideoBlot extends BlockEmbed {
  static create (value) {
    let node = super.create()
    node.setAttribute('src', value.url)
    node.setAttribute('controls', value.controls)
    node.setAttribute('width', value.width)
    node.setAttribute('height', value.height)
    node.setAttribute('webkit-playsinline', true)
    node.setAttribute('playsinline', true)
    node.setAttribute('x5-playsinline', true)
    return node;
  }
 
  static value (node) {
    return {
      url: node.getAttribute('src'),
      controls: node.getAttribute('controls'),
      width: node.getAttribute('width'),
      height: node.getAttribute('height')
    };
  }
}
VideoBlot.blotName = 'simpleVideo'
VideoBlot.tagName = 'video'
Quill.register(VideoBlot)
export default {
  name: "TestQuillEditor",
  mounted () {
    this.initQuill()
  },
  beforeDestroy () {
    this.quill = null
    delete this.quill
  },
  methods: {
    initQuill () {
      const quill = new Quill('#editor', {
        theme: 'snow',
        modules: {
          toolbar: '#toolbar'
        }
      })
      this.quill = quill
      this.quill.getModule("toolbar").addHandler("image", this.uploadImageHandler);
      this.quill.getModule("toolbar").addHandler("video", this.uploadVideoHandler);
    },
    uploadImageSucess (type) {
        const addImageRange = this.quill.getSelection();
        const newRange = 0 + (addImageRange !== null ? addImageRange.index : 0);
        if(type===1){
          const url = 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100';
          this.quill.insertEmbed(newRange, 'image', url);
          this.quill.setSelection(1 + newRange);
       }
       else{
            const url = 'https://www.w3school.com.cn/i/movie.mp4';
            this.quill.insertEmbed(newRange, 'simpleVideo', {url,controls: 'controls',width: '320',height: '240',autoplay:'autoplay'});
            this.quill.setSelection(1 + newRange);
        }
    },
    uploadImageHandler () {
      const input = document.querySelector('#uploadImg')
      input.value = ''
      input.click()
    },  
    uploadImage (event) {
      if (typeof XMLHttpRequest === "undefined") {
        return;
      }
    var xhr = new XMLHttpRequest();
    const formData = new FormData();
    formData.append('upload_file', event.target.files[0]);
      if (typeof XMLHttpRequest === "undefined") {
        return;
      }
      var xhr = new XMLHttpRequest();
      var text = "成功";
      const that= this;
      xhr.onload = function onload() {
        if (xhr.status < 200 || xhr.status >= 300) {
          // return option.onError(this.getError(action, option, xhr));
          text = xhr.responseText || xhr.response;
          alert("失败" + text);
        }
        console.log(text);
        that.uploadImageSucess(1);   
      };

      xhr.open("post", "http://localhost:8088/test", true);     
      xhr.send(formData); 
    },
    uploadVideoHandler () { 
      const input = document.querySelector('#uploadVideo')
      input.value = ''
      input.click()
      },
    uploadVideo (event) { if (typeof XMLHttpRequest === "undefined") {
        return;
      }
    var xhr = new XMLHttpRequest();
    const formData = new FormData();
    formData.append('upload_file', event.target.files[0]);
      if (typeof XMLHttpRequest === "undefined") {
        return;
      }
      var xhr = new XMLHttpRequest();
      var text = "成功";
      const that= this;
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
      xhr.send(formData); }  
  }
}
</script>