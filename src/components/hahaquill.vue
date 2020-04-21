<template>
  <div>
    <!-- <div id="toolbar">
      <span class="ql-formats">
        <button class="ql-image">image123</button>
        <button class="ql-video">video</button>
      </span>
    </div> -->
    <div id="editor"></div>
    <!-- 测试video标签 -->
    <!-- <video src="https://www.w3school.com.cn/i/movie.mp4" controls="controls" width="100%" height="100%" webkit-playsinline="true" playsinline="true" x5-playsinline="true"></video> -->
    <input id="uploadImg" ref="uploadImg" type="file" style="display:none" accept="image/jpeg, image/png"
      @change="uploadImage">
    <input id="uploadVideo" ref="uploadVideo" type="file" style="display:none" accept="video/*" @change="uploadVideo">
  </div>
</template>
 
<script>
import Quill from 'quill'

//视频处理
const BlockEmbed = Quill.import('blots/block/embed')
class VideoBlot extends BlockEmbed {
  static create(value) {
    let node = super.create()
    node.setAttribute('src', value.url)
    node.setAttribute('controls', value.controls)
    node.setAttribute('width', value.width)
    node.setAttribute('height', value.height)
    node.setAttribute('webkit-playsinline', true)
    node.setAttribute('playsinline', true)
    node.setAttribute('x5-playsinline', true)
    return node
  }

  static value(node) {
    return {
      url: node.getAttribute('src'),
      controls: node.getAttribute('controls'),
      width: node.getAttribute('width'),
      height: node.getAttribute('height')
    }
  }
}

VideoBlot.blotName = 'simpleVideo'
VideoBlot.tagName = 'video'
Quill.register(VideoBlot)

var toolbarOptions = [
  ['bold', 'italic', 'underline', 'strike'], // toggled buttons
  ['blockquote', 'code-block'],
  [{ header: 1 }, { header: 2 }], // custom button values
  [{ list: 'ordered' }, { list: 'bullet' }],
  [{ script: 'sub' }, { script: 'super' }], // superscript/subscript
  [{ indent: '-1' }, { indent: '+1' }], // outdent/indent
  [{ direction: 'rtl' }], // text direction
  [{ size: ['small', false, 'large', 'huge'] }], // custom dropdown
  [{ header: [1, 2, 3, 4, 5, 6, false] }],
  [{ color: [] }, { background: [] }], // dropdown with defaults from theme
  [{ font: [] }],
  [{ align: [] }],
  ['link', 'image', 'video', 'formula'],
  ['clean'] // remove formatting button
]

export default {
  name: 'myquill',
  props: {
    //富文本内容
    content: {
      type: String,
      default: '编辑器默认值'
    },
    //文件大小阈值，单位字节B，大于1M=1024B
    threshold: {
      type: Number,
      default: 1025
    },
    //宽度
    width: {
      type: Number,
      default: 1080
    },
    //高度
    height: {
      type: Number,
      default: 100
    },
    //高度
    quality: {
      type: Number,
      default: 0.2
    }
  },

  data() {
    return {
      contentchange: '',
      condenseBase64: ''
    }
  },

  watch: {
    content(newvalue, oldvalue) {
      //   this.quill.root.innerHTML = this.content //初始化赋值
      this.quill.root.innerHTML = newvalue //初始化赋值
      //   this.quill.updateContents(new Delta().retain(6).delete(5).insert('Quill啊哈哈哈').retain(1, { bold: true }));
    },
    contentchange(newvalue, oldvalue) {
      this.$emit('update:content', newvalue)
    }
  },

  mounted() {
    this.initQuill()
  },

  beforeDestroy() {
    this.quill = null
    delete this.quill
  },
  methods: {
    initQuill() {
      const quill = new Quill('#editor', {
        theme: 'snow',
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
        placeholder: '我的编辑器占位符...'
        // readOnly: true,
      })
      this.quill = quill
      this.quill.getModule('toolbar').addHandler('image', this.uploadImageHandler)
      this.quill.getModule('toolbar').addHandler('video', this.uploadVideoHandler)
      //   this.quill.setContents(this.content)
      this.quill.root.innerHTML = this.content //初始化赋值
      //   this.quill.updateContents(new Delta().retain(6).delete(5).insert('Quill啊哈哈哈').retain(1, { bold: true }));
      const that = this
      //   this.quill.on('editor-change', function(eventName, ...args) {
      //     if (that.quill.root) {
      //       that.contentchange = that.quill.root.innerHTML
      //       //   let length1 = that.quill.getSelection().index
      //       let length = that.quill.selection.savedRange.index
      //       that.quill.setSelection(1 + length)
      //       console.log(
      //         '$ editor-change 888 that.quill.selection.savedRange.index:{that.quill.selection.savedRange.index}'
      //       )
      //     }
      //   })
      this.quill.on('blur', function(eventName, ...args) {
        if (that.quill.root) {
          that.contentchange = that.quill.root.innerHTML
          //   let length1 = that.quill.getSelection().index
          let length = that.quill.selection.savedRange.index
          that.quill.setSelection(1 + length)
          console.log(
            '$ editor-change 888 that.quill.selection.savedRange.index:{that.quill.selection.savedRange.index}'
          )
        }
      })
    },

    uploadImageHandler() {
      const input = document.querySelector('#uploadImg')
      input.value = ''
      input.click()
      // this.$refs.uploadImage.value = "";
      // this.$refs.uploadImage.click();
    },

    //el-upload文件上传的处理逻辑
    beforeUpload(file) {
      const isJPGorPNG = file.type === 'image/jpeg' || file.type === 'image/png'
      const isLessthan2M = file.size / 1024 / 1024 < 2 //最大限制2M

      if (!isJPGorPNG) {
        this.$message.error('上传图片只能是 JPG,PNG 格式!')
      }
      if (!isLessthan2M) {
        this.$message.error('上传图片大小不能超过 2MB!')
      }
      return isJPGorPNG && isLessthan2M
    },
    uploadImage(event) {
      var file = event.target.files[0]
      const that = this
      if (that.beforeUpload(file)) {
        //上传图片大于1M进行压缩
        if (file.size / 1024 > that.threshold) {
          this.condenseFile(file, function(base64Codes) {
            that.uploadImageSucess(1, base64Codes)
          })
        } else {
          this.convertBase64Url(file)
        }
        this.uploadImageSucess(1, this.condenseBase64)
      }
    },

    uploadVideoHandler() {
      const input = document.querySelector('#uploadVideo')
      input.value = ''
      input.click()
      // this.$refs.uploadVideo.value = "";
      // this.$refs.uploadVideo.click();
    },

    uploadVideo(event) {
      if (typeof XMLHttpRequest === 'undefined') {
        return
      }
      var xhr = new XMLHttpRequest()
      const formData = new FormData()
      //   formData.append('upload_file', event.target.files[0])
      var text = '成功'
      const that = this
      xhr.onload = function onload() {
        if (xhr.status < 200 || xhr.status >= 300) {
          // return option.onError(this.getError(action, option, xhr));
          text = xhr.responseText || xhr.response
          alert('失败' + text)
        }
        // console.log(text)
        that.uploadImageSucess(2, 'https://www.w3school.com.cn/i/movie.mp4')
      }

      xhr.open('post', 'http://localhost:8088/test', true)
      xhr.send(formData)
    },

    uploadImageSucess(type, url) {
      //   const addImageRange = this.quill.getSelection()
      //   const newRange = 0 + (addImageRange !== null ? addImageRange.index : 0)
      let newRange = this.quill.selection.savedRange.index

      if (type === 1) {
        // const url =
        //   'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'
        this.quill.insertEmbed(newRange, 'image', url)
        this.quill.setSelection(1 + newRange)
        console.log(
          '$ uploadImageSucess 7777 his.quill.selection.savedRange.index:{this.quill.selection.savedRange.index}'
        )
      } else {
        // const url = 'https://www.w3school.com.cn/i/movie.mp4'
        this.quill.insertEmbed(newRange, 'simpleVideo', {
          url,
          controls: 'controls',
          width: '320',
          height: '240',
          autoplay: 'autoplay'
        })
        this.quill.setSelection(1 + newRange)
      }
      //获取内容
      //   console.log(this.quill.getContents())
      //   console.log(this.quill.getText())
      //   var html = this.quill.root.innerHTML
      // alert(html);
      //   console.log(html)
      //   var xx = this.quillGetHTML(this.quill.getContents())
      //  alert(xx);
      //   console.log(xx)
    },

    //获取富文本信息
    quillGetHTML(inputDelta) {
      //护球编辑器的内容
      var tempCont = document.createElement('div')
      new Quill(tempCont).setContents(inputDelta)
      return tempCont.getElementsByClassName('ql-editor')[0].innerHTML
    },

    //文件转换成base64字符串
    convertBase64Url(file) {
      var ready = new FileReader()
      ready.readAsDataURL(file)
      const that = this
      ready.onload = function() {
        var fileResult = this.result
        that.uploadImageSucess(1, fileResult)
      }
    },

    //压缩文件第一步
    condenseFile(file, callback) {
      var ready = new FileReader()
      ready.readAsDataURL(file)
      const that = this
      ready.onload = function() {
        var fileResult = this.result
        that.condenseCanvasDataURL(fileResult, callback)
      }
    },

    //压缩文件第二步，重新绘制图片，并返回压缩以后的文件的base64的值，可以把base64的值作为参数传给回调接口
    condenseCanvasDataURL(path, callback) {
      var img = new Image()
      img.src = path
      const that1 = this
      img.onload = function() {
        var that = this
        //默认压缩后图片规格
        var quality = 0.5
        var w = that.width
        var h = that.height
        var scale = w / h
        //计算图片的实际大小，设置宽高比例
        w = w > that1.width ? that1.width : w
        h = w / scale
        if (that1.quality && that1.quality > 0 && that1.quality <= 1) {
          quality = that1.quality
        }

        //生成canvas
        var canvas = document.createElement('canvas')
        var ctx = canvas.getContext('2d')
        // 创建属性节点
        var anw = document.createAttribute('width')
        anw.nodeValue = w
        var anh = document.createAttribute('height')
        anh.nodeValue = h
        canvas.setAttributeNode(anw)
        canvas.setAttributeNode(anh)
        ctx.drawImage(that, 0, 0, w, h)

        var base64 = canvas.toDataURL('image/jpeg', quality)
        // 回调函数返回压缩以后的文件的base64的值
        callback(base64)
      }
    },

    condenseconvertBase64UrlToBlob(urlData) {
      var arr = urlData.split(','),
        mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]),
        n = bstr.length,
        u8arr = new Uint8Array(n)
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n)
      }
      return new Blob([u8arr], { type: mime })
    },

    //压缩文件第一步
    uploadFile(event) {
      if (typeof XMLHttpRequest === 'undefined') {
        return
      }
      var xhr = new XMLHttpRequest()
      if (xhr.upload) {
        xhr.upload.onprogress = function progress(e) {
          if (e.total > 0) {
            e.percent = (e.loaded / e.total) * 100
          }
          //   option.onProgress(e);吧上传进度回调给其他的接口
        }
      }
      const formData = new FormData()
      //   formData.append('upload_file', event.target.files[0]);
      var file = event.target.files[0]
      const that = this
      //上传图片大于1M进行压缩
      if (file.size / 1024 > 1025) {
        that.photoCompress(file, { quality: 0.2 }, function(base64Codes) {
          var bl = that.convertBase64UrlToBlob(base64Codes)
          formData.append('files', bl, file.name)
        })
      } else {
        formData.append('files', file, file.name)
      }
      xhr.onerror = function error(e) {
        alert('失败' + e)
      }
      var text = '成功'
      xhr.onload = function onload() {
        if (xhr.status < 200 || xhr.status >= 300) {
          text = xhr.responseText || xhr.response
          alert('失败' + text)
        }
        // console.log(text)
        //上传成功，根据xhr.response的返回信息中的url，将url插入到编辑区
        that.uploadImageSucess(1)
      }

      xhr.open('post', 'http://localhost:8084/test', true)
      xhr.send(formData)
    },
    //图片压缩处理 压缩文件第一步
    photoCompress(file, objCompressed, objDiv) {
      var ready = new FileReader()
      ready.readAsDataURL(file)
      const that = this
      ready.onload = function() {
        var fileResult = this.result
        that.canvasDataURL(fileResult, objCompressed, objDiv)
      }
    },
    //压缩文件第二步
    canvasDataURL(path, objCompressed, callback) {
      var img = new Image()
      img.src = path
      img.onload = function() {
        var that = this
        //默认压缩后图片规格
        var quality = 0.5
        var w = that.width
        var h = that.height
        var scale = w / h
        //实际要求
        w = objCompressed.width || w
        h = objCompressed.height || w / scale
        if (objCompressed.quality && objCompressed.quality > 0 && objCompressed.quality <= 1) {
          quality = objCompressed.quality
        }

        //生成canvas
        var canvas = document.createElement('canvas')
        var ctx = canvas.getContext('2d')
        // 创建属性节点
        var anw = document.createAttribute('width')
        anw.nodeValue = w
        var anh = document.createAttribute('height')
        anh.nodeValue = h
        canvas.setAttributeNode(anw)
        canvas.setAttributeNode(anh)
        ctx.drawImage(that, 0, 0, w, h)

        var base64 = canvas.toDataURL('image/jpeg', quality)
        // 回调函数返回base64的值
        callback(base64)
      }
    },
    //上传文件压缩的处理完成之后，回调方法 压缩文件第三步
    convertBase64UrlToBlob(urlData) {
      var arr = urlData.split(','),
        mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]),
        n = bstr.length,
        u8arr = new Uint8Array(n)
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n)
      }
      return new Blob([u8arr], { type: mime })
    }
  }
}
</script>