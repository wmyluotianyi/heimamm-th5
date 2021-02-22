<template>
  <div class='container'>
     <el-card>
       <!-- 头部-按钮 -->
      <div class='title_btn'>
        <span style="font-size:12px;color:pink">说明：目前支持学科和关键字条件筛选</span>
        <el-button type="success" icon="el-icon-edit" @click="$router.push('new')">新增试题</el-button>
      </div>
      <!-- 筛选表单 -->
      <el-form label-width="80px" size="small">
        <el-row>
          <el-col :span="6">
            <el-form-item label="学科">
              <el-select v-model="requestParams.subjectID" placeholder="请选择" @change="changeSubject(requestParams.subjectID)">
                <el-option v-for="obj in subjectOptions" :key="obj.value" :label="obj.label" :value="obj.value"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="二级目录">
              <el-select v-model="requestParams.catalogID" placeholder="请选择">
                <el-option v-for="obj in directoryOptions" :key="obj.value" :label="obj.label" :value="obj.value"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="标签">
              <el-select v-model="requestParams.tags" placeholder="请选择">
                <el-option v-for="obj in tagOptions" :key="obj.value" :label="obj.label" :value="obj.value"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="关键字">
              <el-input placeholder="根据题干搜索" v-model="requestParams.keyword"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="试题类型">
              <el-select v-model="requestParams.questionType" placeholder="请选择">
                <el-option v-for="obj in questionTypeOptions" :key="obj.value" :label="obj.label" :value="obj.value"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="难度">
              <el-select v-model="requestParams.difficulty" placeholder="请选择">
                <el-option v-for="obj in diffOptions" :key="obj.value" :label="obj.label" :value="obj.value"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="方向">
              <el-select v-model="requestParams.direction" placeholder="请选择">
                <el-option v-for="item in directionOptions" :key="item" :label="item" :value="item"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="录入人">
              <el-select v-model="requestParams.creatorID" placeholder="请选择">
                <el-option v-for="obj in userOptions" :key="obj.id" :label="obj.username" :value="obj.id"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
           <el-col :span="6">
            <el-form-item label="题目备注">
              <el-input v-model="requestParams.remarks"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="企业简称">
              <el-input v-model="requestParams.shortName"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="城市">
              <el-select v-model="requestParams.province" style="width:48%;margin-right:2%" @change="handleCity(requestParams.province)">
                <el-option v-for="item in cityOptions" :key="item" :label="item" :value="item"></el-option>
              </el-select>
              <el-select v-model="requestParams.city" style="width:50%">
                <el-option v-for="item in areaOptions" :key="item" :label="item" :value="item"></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item style="text-align:right">
              <el-button @click="clear()">清除</el-button>
              <el-button type="primary" @click="filter()">搜索</el-button>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <el-alert
        :title="`数据一共 ${total} 条`"
        style="margin-bottom:15px"
        type="info"
        :closable="false"
        show-icon
      ></el-alert>
      <el-table :data="questions">
        <el-table-column label="试题编号" prop="number" width="120px"></el-table-column>
        <el-table-column label="学科" prop="subject"></el-table-column>
        <el-table-column label="目录" prop="catalog"></el-table-column>
        <el-table-column label="题型">
          <template slot-scope="scope">
            {{ questionType.find(item=>item.value===parseInt(scope.row.questionType)).label }}
          </template>
        </el-table-column>
        <el-table-column label="题干" width="280px">
          <template slot-scope="scope">
            <div v-html="scope.row.question"></div>
          </template>
        </el-table-column>
        <el-table-column label="录入时间" width="180px">
          <template slot-scope="scope">
            {{scope.row.addDate|parseTimeByString}}
          </template>
        </el-table-column>
        <el-table-column label="难度">
          <template slot-scope="scope">
            {{difficulty.find(item=>item.value===parseInt(scope.row.difficulty)).label}}
          </template>
        </el-table-column>
        <el-table-column label="录入人" prop="creator"></el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <el-button plain type="primary" size="small" circle icon="el-icon-view" title="预览" @click="openPreviewDialog(scope.row)"></el-button>
            <el-button plain type="success" size="small" circle icon="el-icon-edit" title="修改" @click="$router.push(`new?id=${scope.row.id}`)"></el-button>
            <el-button plain type="danger" size="small" circle icon="el-icon-delete" title="删除" @click="delQuestion(scope.row)"></el-button>
            <el-button plain type="warning" size="small" circle icon="el-icon-check" title="加入精选" @click="addChoice(scope.row)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        style="margin-top:20px;text-align:right"
        background
        layout="prev, pager, next, sizes, jumper"
        :total="total"
        :page-size="requestParams.pagesize"
        :current-page="requestParams.page"
        @current-change="pager"
        :page-sizes="[5,10,20,50]"
        @size-change="handleSizeChange"
      ></el-pagination>
    </el-card>
    <questions-preview ref="questionPreview" :data="questionInfo"></questions-preview>
  </div>
</template>

<script>
import { simple as subjectList } from '@/api/hmmm/subjects.js'
import { simple as directoryList } from '@/api/hmmm/directorys'
import { simple as tagList } from '@/api/hmmm/tags'
import { questionType, difficulty, direction } from '@/api/hmmm/constants'
import { simple as userList } from '@/api/base/users'
import { provinces as getCity, citys as getArea } from '@/api/hmmm/citys'
import { list as questionList, remove as questionDel, choiceAdd } from '@/api/hmmm/questions'
import QuestionsPreview from '../components/questions-preview'
export default {
  name: 'questions',
  components: {
    QuestionsPreview
  },
  data () {
    return {
      total: 0,
      questionType,
      difficulty,
      requestParams: {
        subjectID: null,
        catalogID: null,
        tags: null,
        keyword: null,
        questionType: null,
        difficulty: null,
        direction: null,
        creatorID: null,
        remarks: null,
        shortName: null,
        province: null,
        city: null,
        page: 1,
        pagesize: 5
      },
      // 学科
      subjectOptions: [],
      // 二级目录
      directoryOptions: [],
      // 标签
      tagOptions: [],
      // 试题类型
      questionTypeOptions: questionType,
      // 难度
      diffOptions: difficulty,
      // 方向
      directionOptions: direction,
      // 录入人
      userOptions: [],
      // 城市
      cityOptions: getCity(),
      // 地区
      areaOptions: [],
      // 表格数据
      questions: [],
      questionInfo: {}
    }
  },
  async created () {
    // 获取学科下拉列表
    const res = await subjectList()
    this.subjectOptions = res.data
    // 获取用户下拉列表
    const result = await userList()
    this.userOptions = result.data
    // console.log(this.questionType)
    // console.log(this.requestParams.questionType)
    // 加载数据列表
    this.getList()
  },
  methods: {
    async changeSubject (subjectID) {
      this.requestParams.catalogID = null
      this.requestParams.tags = null
      if (subjectID) {
        // 获取二级目录下拉列表
        const res = await directoryList({ subjectID })
        this.directoryOptions = res.data
        // 获取标签下拉列表
        const result = await tagList({ subjectID })
        this.tagOptions = result.data
      } else {
        this.directoryOptions = []
        this.tagOptions = []
      }
    },
    handleCity (cityName) {
      this.requestParams.city = null
      this.areaOptions = getArea(cityName)
    },
    clear () {
      for (const key in this.requestParams) {
        this.requestParams[key] = null
      }
    },
    async getList () {
      const res = await questionList(this.requestParams)
      // console.log(res)
      this.questions = res.data.items
      this.total = res.data.counts
    },
    handleSizeChange (size) {
      this.requestParams.page = 1
      this.requestParams.pagesize = size
      this.getList()
    },
    pager (np) {
      this.requestParams.page = np
      this.getList()
    },
    filter () {
      this.requestParams.page = 1
      this.getList()
    },
    openPreviewDialog (questionInfo) {
      this.questionInfo = questionInfo
      // console.log(this.questionInfo)
      // 在created()钩子函数执行的时候DOM 其实并未进行任何渲染，而此时进行DOM操作并无作用，而在created()里使用this.$nextTick()可以等待dom生成以后再来获取dom对象
      this.$nextTick(() => {
        this.$refs.questionPreview.open(this.questionInfo.id)
      })
    },
    async delQuestion (question) {
      await this.$confirm('此操作将永久删除该题目, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
      await questionDel(question)
      this.$message.success('删除成功')
      this.getList()
    },
    async addChoice (question) {
      await this.$confirm('此操作将该题目加入精选, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'info'
      })
      await choiceAdd({ id: question.id, choiceState: 1 })
      this.$message.success('加入精选成功')
      this.getList()
    }
  }
}
</script>

<style scoped lang='scss'>
.container {
  padding: 0 10px;
  margin: 10px 0;
}
.title_btn {
  margin-bottom: 15px;
  display: flex;
  justify-content: space-between;
}
</style>
