<template>
  <div>
    <quill-editor
    v-model="content"
    :options="editorOption"
    ref="QuillEditor">
</quill-editor></div>
</template>

<script>
//QuillEditor富文本编辑器
import { quillEditor }  from 'vue-quill-editor'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
 // 工具栏配置
    const toolbarOptions = [
      ['bold', 'italic', 'underline', 'strike'],        // toggled buttons
      ['blockquote', 'code-block'],
    
      [{'header': 1}, {'header': 2}],               // custom button values
      [{'list': 'ordered'}, {'list': 'bullet'}],
      [{'script': 'sub'}, {'script': 'super'}],      // superscript/subscript
      [{'indent': '-1'}, {'indent': '+1'}],          // outdent/indent
      [{'direction': 'rtl'}],                         // text direction
    
      [{'size': ['small', false, 'large', 'huge']}],  // custom dropdown
      [{'header': [1, 2, 3, 4, 5, 6, false]}],
    
      [{'color': []}, {'background': []}],          // dropdown with defaults from theme
      [{'font': []}],
      [{'align': []}],
      ['link', 'image', 'video'],
      ['clean']                                         // remove formatting button
    ]
export default {
  name: 'quilleditor',
  props: {
    content: {
      type: String,
      default: 'quill编辑器默认值'
    },
    editorOption:{
        modules: {
            toolbar: {
                container: toolbarOptions,  // 工具栏
                handlers: {
                          'image': function (value) {
                                    if (value) {
                                        alert('自定义图片')
                                    } else {
                                        // this.quill.format('image', false);
                                        QuillEditor.format('image', false);
                                    }
                                }
                            }
                        }
            }
    }
  }
}
</script>

<style>

</style>