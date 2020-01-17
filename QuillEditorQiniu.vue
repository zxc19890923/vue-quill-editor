<template>
  <div id='quillEditorQiniu'>
    <el-row v-loading="loadingImg">
      <quill-editor
        :style="{width: width ? width: '100%',  background: '#fff'}"
        v-model="content"
        ref="myQuillEditor"
        :options="editorOption"
        @change="onEditorChange($event)"
      >
      </quill-editor>
    </el-row>
    <Upload
      class="avatar-uploader"
      :accept="'image/*'"
      :action="'https://upload.qiniup.com'"
      name='file'
      :data="updateParams"
      :show-upload-list="false"
      :on-success="uploadSuccess"
      :on-error="uploadError"
      :before-upload="beforeUpload"
    >
    </Upload>
  </div>
</template>
<script>
import * as Quill from 'quill'
import { quillEditor } from 'vue-quill-editor'
import { ImageDrop } from 'quill-image-drop-module'
import ImageResize from 'quill-image-resize-module'
import {ImageExtend} from 'quill-image-paste-module'
let fontSizeStyle = Quill.import('attributors/style/size')
fontSizeStyle.whitelist = ['12px', '14px', '16px', '20px', '24px', '36px']
Quill.register(fontSizeStyle, true)
Quill.register('modules/imageDrop', ImageDrop)
Quill.register('modules/imageResize', ImageResize)
Quill.register('modules/ImageExtend', ImageExtend)
const toolbarOptions = [
  ['bold', 'italic', 'underline', 'strike'],
  [{'color': ['#000000', '#e60000', '#ff9900', '#ffff00', '#008a00', '#0066cc', '#9933ff', '#ffffff', '#facccc', '#ffebcc', '#ffffcc', '#cce8cc', '#cce0f5', '#ebd6ff', '#bbbbbb', '#f06666', '#ffc266', '#ffff66', '#66b966', '#66a3e0', '#c285ff', '#888888', '#a10000', '#b26b00', '#b2b200', '#006100', '#0047b2', '#6b24b2', '#444444', '#5c0000', '#663d00', '#666600', '#003700', '#002966', '#3d1466']}, { 'size': fontSizeStyle.whitelist }],
  [{ list: 'ordered' }, { list: 'bullet' }],
  [{ indent: '-1' }, { indent: '+1' }],
  ['link', 'image']
]
// 自定义编辑器的工作条
export default {
  name: 'quill-editor-qiniu',
  components: {
    quillEditor,
    ImageResize
  },
  props: ['initValue', 'width', 'placeholder'],
  created () {
    // 获取初始化回显内容
    this.content = this.initValue
    // 获取上传token
    this.$store.dispatch('uploadToken')
  },
  mounted () {
    // 工具栏中的图片图标被单击的时候调用这个方法
    let imgHandler = (state) => {
      if (state) {
        document.querySelector('.avatar-uploader input').click()
      }
    }
    // 当工具栏中的图片图标被单击的时候
    this.$refs.myQuillEditor.quill.getModule('toolbar').addHandler('image', imgHandler)
  },
  data () {
    return {
      content: '',
      editorOption: {
        placeholder: this.placeholder ? this.placeholder : '请输入',
        theme: 'snow',
        modules: {
          toolbar: {
            container: toolbarOptions
          },
          // 拖拽上传和调整图片大小
          imageDrop: true,
          imageResize: {
            displayStyles: {
              backgroundColor: 'black',
              border: 'none',
              color: 'white'
            },
            modules: [ 'Resize', 'DisplaySize', 'Toolbar' ]
          },
          // 截屏上传
          ImageExtend: {
            loading: true,
            name: 'file',
            change: (xhr, FormData) => {
              FormData.append('token', this.$store.state.upload_token)
            },
            action: 'https://upload.qiniup.com',
            response: (res) => {
              console.log(res, 'response')
              return this.$store.getters.upload_url + res.key
            }
          }
        }
      },
      updateParams: {},
      loadingImg: false
    }
  },
  watch: {
    initValue: function (newVal, oldVal) {
      this.content = newVal
    }
  },
  methods: {
    onEditorChange (event) {
      console.log(event, 'change')
      this.$emit('getEditorInfo', event)
    },
    beforeUpload (request, file) {
      // 设置上传参数
      this.updateParams.token = this.$store.state.upload_token
      this.updateParams.key = request.name
      this.loadingImg = true
    },
    // 上传图片成功
    uploadSuccess (res, file) {
      // file 返回的文件信息，也可以在这里调用七牛上传。
      console.log(res, file, this.$store.getters.upload_url + res.key, 'success')
      // 上传完成以后修改图片地址，回显到quill编辑器中
      let quill = this.$refs.myQuillEditor.quill
      let length = quill.getSelection() ? quill.getSelection().index : 0
      // 插入图片  res.info为服务器返回的图片地址
      console.log(this.$store.getters.upload_url + file.name)
      // quill.insertEmbed(length, 'image', this.$store.getters.upload_url + file.name)
      quill.insertEmbed(length, 'image', this.$store.getters.upload_url + res.key)
      // 调整光标到最后
      quill.setSelection(length + 1)
      this.loadingImg = false
    },
    // 上传图片失败
    uploadError (error, file, list) {
      console.log(error, file, list, 'error')
      this.loadingImg = false
      if (file.error === 'file exists') {
        this.$message({type: 'error', message: list.name + ' 已存在，请重新选择！'})
      } else {
        this.$message({type: 'error', message: list.name + ' 上传出错，请重新上传！'})
      }
    }
  }
}
</script>
<style scoped>
</style>
