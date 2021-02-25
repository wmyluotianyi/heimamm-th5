<template>
  <div class="container">
    <!-- element-ui的dialog对话框组件 -->
    <!-- visible.sync：双向绑定值。visible是否显示 Dialog，支持 .sync 修饰符 title为dialog的标题-->
    <el-dialog title="预览文章" :visible.sync="isShow">
      <div class="title">
        <h2>{{ data.title }}</h2>
        <div class="info">
          <span>{{ data.createTime|parseTimeByString }}</span>
          <span class="el-icon-s-custom"></span>
          <span>{{ data.username }}</span>
          <span class="el-icon-view"></span>
          <span>{{ data.visits }}</span>
        </div>
      </div>
      <div class="content" v-html="data.articleBody"></div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'articlesPreview',
  data () {
    return {
      isShow: false
    }
  },
  props: { // 父向子传值 default疑惑待解
    data: {
      default: () => {},
      type: Object
    }
  },
  methods: {
    open () {
      this.isShow = true
    }
  }
}
</script>

<style scoped>
.title {
  border-bottom: 1px dashed #ccc;
  padding-bottom: 10px;
}
h2 {
  padding: 0;
  margin: 0;
}
.info {
  margin-top: 15px;
}
.content {
  background: #f5f5f5;
  padding: 10px;
}
.el-icon-s-custom {
    padding: 5px;
    color: red;
}
.el-icon-view {
    padding:5px;
    color:skyblue;
}
</style>
