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
              <el-button type="primary">搜索</el-button>
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
            <!-- {{questionType.find(item=>item.value===+scope.row.questionType).label}} -->
            {{ scope.row.questionType }}
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
            <!-- {{difficulty.find(item=>item.value===+scope.row.difficulty).label}} -->
            {{ scope.row.difficulty }}
          </template>
        </el-table-column>
        <el-table-column label="录入人" prop="creator"></el-table-column>
        <el-table-column label="操作" width="180px">
          <template>
            <el-button plain type="primary" size="small" circle icon="el-icon-view" title="预览"></el-button>
            <el-button plain type="success" size="small" circle icon="el-icon-edit" title="修改"></el-button>
            <el-button plain type="danger" size="small" circle icon="el-icon-delete" title="删除"></el-button>
            <el-button plain type="warning" size="small" circle icon="el-icon-check" title="加入精选"></el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
import { simple as subjectList } from '@/api/hmmm/subjects.js'
import { simple as directoryList } from '@/api/hmmm/directorys'
import { simple as tagList } from '@/api/hmmm/tags'
import { questionType, difficulty, direction } from '@/api/hmmm/constants'
import { simple as userList } from '@/api/base/users'
import { provinces as getCity, citys as getArea } from '@/api/hmmm/citys'
import { list as questionList } from '@/api/hmmm/questions'
export default {
  name: 'questions',
  data () {
    return {
      total: 0,
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
        city: null
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
      questions: []
    }
  },
  async created () {
    // 获取学科下拉列表
    const res = await subjectList()
    this.subjectOptions = res.data
    // 获取用户下拉列表
    const result = await userList()
    this.userOptions = result.data
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
      console.log(res)
      this.questions = res.data.items
      this.total = res.data.counts
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
