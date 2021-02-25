<template>
  <div class="container">
    <el-card>
      <el-row>
        <el-col :span="18">
          <!-- 设置 inline 属性可以让表单域变为行内的表单域 -->
          <el-form label-width="80px" size="small" :inline="true">
            <el-form-item label="关键字">
              <el-input
                placeholder="根据文章标题搜索"
                style="200px"
                v-model="params.keyword"
              ></el-input>
            </el-form-item>
            <el-form-item label="状态">
              <el-select v-model="params.state">
                <el-option :value="1" label="启用"></el-option>
                <el-option :value="0" label="禁用"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item style="text-align: right">
              <el-button @click="clear()">清除</el-button>
              <el-button @click="search()" type="primary">搜索</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="6" style="text-align: right">
          <el-button
          icon="el-icon-edit"
          type="success"
          size="small"
          @click="addArticle()"
            >新增技巧</el-button
          >
        </el-col>
        <!-- 警示框 -->
        <el-alert
          :title="`数据一共 ${total} 条`"
          type="info"
          show-icon
          style="margin-bottom: 15px"
        >
        </el-alert>
      </el-row>

      <!-- 模快二: 表格数据-->

      <el-table :data="articles">
        <el-table-column type="index" width="100" label="序号">
        </el-table-column>

        <el-table-column label="文章标题" width="400">
          <!--  slot-scope="scope" 来取得作用域插槽:data绑定的数据 当前行数据的获取也会用到插槽,scope相当于一行的数据  scope.row相当于当前行的数据对象 -->
          <template slot-scope="scope">
            {{ scope.row.title }}
            <a
              href="#"
              v-if="scope.row.videoURL"
              class="el-icon-video-camera-solid video"
              @click="playFn(scope.row.videoURL)"
            ></a>
          </template>
        </el-table-column>
        <!-- 疑惑/忘了的问题:为什么在这儿使用prop就能获取到数据 -->
        <el-table-column label="阅读数" width="300" prop="visits">
        </el-table-column>

        <el-table-column label="录入人" prop="username"> </el-table-column>

        <!-- parseTimeByString 老师写的过滤器filters -->
        <el-table-column label="录入时间" width="300">
          <template slot-scope="scope">{{
            scope.row.createTime | parseTimeByString
          }}</template>
        </el-table-column>

        <el-table-column label="状态" width="100">
          <template slot-scope="scope">{{
            scope.row.state === 1 ? "已启用" : "已禁用"
          }}</template>
        </el-table-column>

        <el-table-column label="操作" width="200">
          <template slot-scope="scope">

            <el-button type="text" @click="previewArticle(scope.row)">预览</el-button>

            <el-button type="text" @click="StateFn(scope.row)">{{
              scope.row.state === 1 ? "禁用" : "启用"
            }}</el-button>

            <el-button
            type="text"
            :disabled="scope.row.state === 1"
            @click="addArticle(scope.row)"
              >修改</el-button
            >

            <el-button
            type="text"
            :disabled="scope.row.state === 1||scope.row.totals > 0"
            @click="delFn(scope.row)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <el-pagination
        style="margin-top: 20px; text-align: right"
        background
        layout="prev, pager, next, sizes, jumper"
        :total="total"
        :page-size="params.pagesize"
        :current-page="params.page"
        :page-sizes="[5, 10, 15, 20]"
        @current-change="pager"
        @size-change="handleSizeChange"
      >
      </el-pagination>
    </el-card>
    <!-- 新增技巧 -->
    <articles-add ref="articlesAdd" :data="currArticle" @updateList="updateList()"></articles-add>
    <!-- 文章预览 -->
    <articles-preview ref="articlesPreview" :data='currArticle'></articles-preview>
    <!-- 视频预览 -->
    <div class="video-preview" v-if="videoURL">
      <div class="video-close" @click="closevideo()">
        <i class="el-icon-circle-close"></i>
      </div>
      <div class="video-box"><video :src="videoURL" ref="video"></video></div>
    </div>
  </div>
</template>
<script>
import { list, changeState, remove } from '@/api/hmmm/articles.js'
import ArticlesPreview from '../components/articles-preview'
import articlesAdd from '../components/articles-add'

export default {
  name: 'ArticlesList',
  components: { ArticlesPreview, articlesAdd },
  data () {
    return {
      params: {
        keyword: null, // 关键字
        state: null, // 状态
        page: 1, // 当前页码
        pagesize: 10 // 一页显示几条
      },
      total: 0, // 文章总共条数
      articles: [], // 所有的文章数组
      currArticle: {}, // 文章数组里的每一篇文章
      videoURL: null // 视频地址
    }
  },
  //   钩子函数:vue实例对象data和el创建完执行
  created () {
    this.getList()
  },
  methods: {
    // 点击清除按钮清除关键字
    clear () {
      this.params = {
        keyword: null,
        state: null,
        page: 1,
        pagesize: 10
      }
    },
    // 点击搜索按钮，按照用户输入的标签名称和选择的状态回到第一页进行查询，更新列表。
    search () {
      this.params.page = 1
      this.getList()
    },
    // 切换分页,点击第几页就切换到第几页
    pager (num) {
      //   console.log(num)获得当前点击的页数 点击第一页就返回1
      this.params.page = num
      this.getList()
    },
    // 需求:选择条数下拉框，按照选择的条数且回到第一页进行查询，更新列表
    handleSizeChange (val) {
      //   console.log(val)
      this.params.pagesize = val
      this.getList()
    },
    // 获取文章页面
    async getList () {
      const { data } = await list(this.params)
      this.articles = data.items
      //   console.log(this.articles)
      this.total = data.counts
    },
    // 点击关闭图标关闭视频
    closevideo () {
      this.$refs.video.pause() // 这是video标签身上的暂停视频的方法
      this.videoURL = null
    },
    // 点击视频图标播放视频
    playFn (videoUrl) {
      this.videoURL = videoUrl
      this.$nextTick(() => {
        this.$refs.video.play() // 播放视频
      })
    },
    // 点击预览按钮预览文章
    previewArticle (article) {
      this.currArticle = article
      //   console.log(this.currArticle)
      this.$nextTick(() => {
        this.$refs.articlesPreview.open()
      })
    },
    // 点击切换状态
    async StateFn (article) {
      await changeState({
        id: article.id,
        state: article.state
      })
      this.$message.success('操作成功')
      //   状态 1 开启 0 屏蔽
      if (article.state === 1) {
        article.state = 0
      } else {
        article.state = 1
      }
    },
    // 点击删除文章
    async delFn (article) {
      await this.$confirm('此操作将永久删除该文章, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'error' // 一个信息提示的图标类型 还有warning info success error
      })
      await remove(article)
      this.$message.success('删除成功')
      this.getList()
    },
    // 新增技巧
    addArticle (article = {}) {
      this.currArticle = article
      this.$nextTick(() => {
        this.$refs.articlesAdd.open()
      })
    },
    updateList () {
      if (!this.currArticle.id) {
        this.params.page = 1
      }
      this.getList()
    }
  }
}
</script>

<style scoped>
.container {
  padding: 10px;
}
.video {
  color: blue;
  font-size: 22px;
}
.video-preview {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.3);
  overflow: hidden;
  z-index: 9998;
}
.video-box {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 800px;
  height: 600px;
  transform: translate(-50%, -50%);
}
.video-close {
  position: absolute;
  top: 30px;
  left: 50%;
  transform: translate(-50%, 0);
  font-size: 40px;
  color: blue;
  z-index: 9999;
}
video {
  width: 100%;
  height: 100%;
}
</style>
