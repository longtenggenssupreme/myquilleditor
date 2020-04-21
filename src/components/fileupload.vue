<template>
  <el-upload class="upload-demo" drag multiple :file-list="fileList" action="http://localhost:8088/test"
    :on-remove="handleRemove" :before-remove="beforeRemove" :limit="3" :http-request="httpupload"
    :on-exceed="handleExceed" :before-upload="beforeUpload" list-type="picture" :name="uploadname">
    <i class="el-icon-upload"></i>
    <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
    <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
  </el-upload>
</template>

<script>
export default {}
</script>

<style>
</style>