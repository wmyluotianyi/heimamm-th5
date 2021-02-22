<template>
  <div class="question-container">
    <el-card>
      <!-- 按钮 -->
      <div class="btn_wrapper">
        <span style="font-size:12px;color:pink">说明：目前支持学科和关键字条件筛选</span>
        <el-button icon="el-icon-edit" @click="$router.push('new')" type="success" size="small">新增试题</el-button>
      </div>
      <!-- 筛选 -->
      <el-form label-width="80px" size="small">
        <el-row>
          <el-col :span="6">
            <el-form-item label="学科">
              <el-select @change="changeSubject" v-model="requestParams.subjectID" style="width:100%">
                <el-option
                  v-for="item in subjectOptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="二级目录">
              <el-select v-model="requestParams.catalogID" style="width:100%">
                <el-option
                  v-for="item in directoryOptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="标签">
              <el-select v-model="requestParams.tags" style="width:100%">
                <el-option
                  v-for="item in tagOptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
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
              <el-select v-model="requestParams.questionType" style="width:100%">
                <el-option
                  v-for="item in questionTypeOptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="难度">
              <el-select v-model="requestParams.difficulty" style="width:100%">
                <el-option
                  v-for="item in diffOptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="方向">
              <el-select v-model="requestParams.direction" style="width:100%">
                <el-option
                  v-for="item in directionOptions"
                  :key="item"
                  :label="item"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="录入人">
              <el-select v-model="requestParams.creatorID" style="width:100%">
                <el-option
                  v-for="item in userOptions"
                  :key="item.id"
                  :label="item.username"
                  :value="item.id"
                ></el-option>
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
              <!-- 城市 -->
              <el-select @change="handleCity" v-model="requestParams.province" style="width:48%;margin-right:2%">
                <el-option
                  v-for="item in cityOptions"
                  :key="item"
                  :label="item"
                  :value="item"
                ></el-option>
              </el-select>
              <!-- 地区 -->
              <el-select v-model="requestParams.city" style="width:50%">
                <el-option
                  v-for="item in areaOptions"
                  :key="item"
                  :label="item"
                  :value="item"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item style="text-align:right">
              <el-button @click="clear()">清除</el-button>
              <el-button @click="filter()" type="primary">搜索</el-button>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <!-- tab切换 -->
      <el-tabs v-model="requestParams.chkState" type="card" @tab-click="changeTab">
        <el-tab-pane label="全部" name="all"></el-tab-pane>
        <el-tab-pane label="待审核" name="0"></el-tab-pane>
        <el-tab-pane label="已审核" name="1"></el-tab-pane>
        <el-tab-pane label="已拒绝" name="2"></el-tab-pane>
      </el-tabs>
      <!-- 数据记录 -->
      <el-alert
        :title="`数据一共 ${total} 条`"
        style="margin-bottom:15px"
        type="info"
        class="alert"
        :closable="false"
        show-icon
      ></el-alert>
      </el-card>
  </div>
</template>

<script>
import { simple as subjectList } from '@/api/hmmm/subjects'
import { simple as directoryList } from '@/api/hmmm/directorys'
import { simple as userList } from '@/api/base/users'
import { difficulty, questionType, direction } from '@/api/hmmm/constants'
import { simple as tagList } from '@/api/hmmm/tags'
import { provinces as getCity, citys as getArea } from '@/api/hmmm/citys'
import { choice as questionList } from '@/api/hmmm/questions'
export default {
  filters: {
    state_chk (val) {
      if (val === 0) return '待审核'
      if (val === 1) return '已审核'
      if (val === 2) return '已拒绝'
    },
    state_pub (row) {
      if (row.publishState === 1 && row.chkState === 1) {
        return '已发布'
      }
      if (row.publishState === 0 && row.chkState === 1) {
        return '已下架'
      }
      return '待发布'
    }
  },
  data () {
    return {
      questionInfo: {},
      questionType,
      difficulty,
      total: 0,
      // 注意：如果响应响应式数据 先声明
      questions: [],
      requestParams: {
        subjectID: null,
        difficulty: null,
        questionType: null,
        tags: null,
        province: null,
        city: null,
        keyword: null,
        remarks: null,
        shortName: null,
        direction: null,
        creatorID: null,
        catalogID: null,
        page: 1,
        pagesize: 5,
        chkState: 'all'
      },
      // 学科选项
      subjectOptions: [],
      // 难度选项
      diffOptions: difficulty,
      // 试题类型选项
      questionTypeOptions: questionType,
      // 标签选项
      tagOptions: [],
      // 城市
      cityOptions: getCity(),
      // 地区 当选择了城市的时候赋值
      areaOptions: [],
      // 方向选项
      directionOptions: direction,
      // 录入人
      userOptions: [],
      // 目录
      directoryOptions: []
    }
  },
  async created () {
    // 学科下拉选项
    const { data: subjectArr } = await subjectList()
    this.subjectOptions = subjectArr
    // 用户
    const { data: userArr } = await userList()
    this.userOptions = userArr
    this.getList()
  },
  methods: {
    // 切换tab
    changeTab () {
      this.requestParams.page = 1
      this.getList()
    },
    // 选择学科查询目录和标签的下拉选项数据
    async changeSubject (subjectID) {
      this.requestParams.catalogID = null
      this.requestParams.tags = null
      if (subjectID) {
        // 获取目录
        const { data: directoryArr } = await directoryList({ subjectID })
        this.directoryOptions = directoryArr
        // 标签下拉选项
        const { data: tagArr } = await tagList({ subjectID })
        this.tagOptions = tagArr
      } else {
        this.directoryOptions = []
        this.tagOptions = []
      }
    },
    clear () {
      for (const key in this.requestParams) {
        if (key !== 'page' && key !== 'pagesize') this.requestParams[key] = null
      }
    },
    filter () {
      this.requestParams.page = 1
      this.getList()
    },
    async getList () {
      const params = { ...this.requestParams }
      if (params.chkState === 'all') params.chkState = null
      const { data: questionsData } = await questionList(params)
      this.questions = questionsData.items
      this.total = questionsData.counts
    },
    handleCity (cityName) {
      this.requestParams.city = null
      this.areaOptions = getArea(cityName)
    }
  }
}
</script>

<style lang="scss" scoped>
.question-container {
  padding: 0 10px;
  margin: 10px 0;
}
.btn_wrapper {
  margin-bottom: 15px;
  display: flex;
  justify-content: space-between;
}
.fs{
  font-size: 12px;
}
</style>
