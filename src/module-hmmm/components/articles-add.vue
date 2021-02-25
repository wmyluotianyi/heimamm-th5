<template>
  <div class="articles-add">
    <el-dialog :title="`${data.id?'修改':'新增'}文章`" :visible.sync="isShow" width="800px">
      <el-form
        ref="form"
        size="small"
        label-width="100px"
        :model="articlesParams"
        :rules="articlesRules"
      >
        <el-form-item label="文章标题:" prop="title">
          <el-input
            v-model="articlesParams.title"
            placeholder="请输入文章标签"
          ></el-input>
        </el-form-item>

        <el-form-item label="文章内容: " prop="articleBody">
          <!-- quill富文本编辑器 -->
          <quill-editor
            v-model="articlesParams.articleBody"
            :options="editorOptions"
          ></quill-editor>
        </el-form-item>

        <el-form-item label="视频地址: " prop="videoURL">
          <el-input
            placeholder="请输入视频地址"
            v-model="articlesParams.videoURL"
          ></el-input>
        </el-form-item>
      </el-form>

      <div slot="footer">
        <el-button @click="isShow = false">取 消</el-button>
        <el-button type="primary" @click="yesFn()">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { add, update } from '@/api/hmmm/articles'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
import { quillEditor } from 'vue-quill-editor'
export default {
  name: 'articlesAdd',
  components: { quillEditor },
  props: {
    data: {
      default: () => {},
      type: Object
    }
  },
  data () {
    return {
      isShow: false,
      articlesParams: {
        id: null,
        title: null,
        articleBody: null,
        videoURL: null
      },
      // 验证规则
      articlesRules: {
        title: [
          { required: true, message: '文章标题不能为空', tirgger: 'blur' }
        ],
        articleBody: [
          { required: true, message: '文章内容不能为空', tirgger: 'blur' }
        ]
      },
      editorOptions: {
        placeholder: '',
        modules: {
          toolbar: [
            ['bold', 'italic', 'underline', 'strike'],
            [{ list: 'ordered' }, { list: 'bullet' }],
            ['blockquote'],
            ['code-block', 'image', 'link']
          ]
        }
      }
    }
  },
  methods: {
    open () {
      this.isShow = true
      if (this.data.id) {
        const { id, title, articleBody, videoURL } = this.data
        this.articlesParams = { id, title, articleBody, videoURL }
      } else {
        this.articlesParams = {
          id: null,
          title: null,
          articleBody: null,
          videoURL: null
        }
      }
      // 清除校验效果clearValidate()element-UI的清除表单验证的方法
      this.$nextTick(() => {
        this.$refs.form.clearValidate()
      })
    },
    // 点击确认 新增文章  提交到页面上
    async yesFn () {
      this.$refs.form.validate(async valid => {
        if (valid) {
          // 提交
          if (!this.data.id) {
            // 新增
            await add(this.articlesParams)
          } else {
            // 修改
            this.articlesParams.id = this.data.id
            await update(this.articlesParams)
          }
          this.$message.success('操作成功')
          this.isShow = false // 隐藏页面
          this.$emit('updateList')
        }
      })
    }
  }
}
</script>

<style scoped >
.articles-add ::v-deep .ql-editor {
  height: 200px;
}
.articles-add ::v-deep .ql-toolbar.ql-snow {
  padding: 0 8px;
}
</style>
