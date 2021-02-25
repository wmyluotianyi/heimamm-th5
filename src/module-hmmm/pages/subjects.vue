<template>
  <div class="container">
    <el-card class="box-card">
      <!-- header -->
      <el-row class="header">
        <el-col :span="18">
          <el-form
            label-width="80px"
            :model="reqParams"
            ref="subjectRef"
            size="small"
            :inline="true"
          >
            <el-form-item label="学科名称">
              <el-input
                style="200px"
                v-model="reqParams.subjectName"
              ></el-input>
            </el-form-item>
            <el-form-item style="text-align: right">
              <el-button @click="clear">清除</el-button>
              <el-button type="primary" @click="getSubjectsList"
                >搜索</el-button
              >
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="6" style="text-align: right">
          <el-button
            icon="el-icon-edit"
            type="success"
            size="small"
            @click="dialogVisible = true"
            >新增学科</el-button
          >
        </el-col>
      </el-row>
      <!-- 总的数据量 -->
      <el-alert
        :title="`数据一共${total}条`"
        style="margin-bottom: 15px"
        type="info"
        class="alert"
        :closable="false"
        show-icon
      >
      </el-alert>
      <!-- table表格 -->
      <el-table :data="dataList">
        <el-table-column label="序号" type="index"></el-table-column>
        <el-table-column label="学科名称" prop="subjectName"></el-table-column>
        <el-table-column label="创建者" prop="username"></el-table-column>
        <el-table-column label="创建日期" width="160px">
          <template slot-scope="scope">
            {{ scope.row.addDate }}
          </template>
        </el-table-column>
        <el-table-column label="前台是否显示">
          <template slot-scope="scope">
            {{ scope.row.isFrontDisplay === 1 ? "是" : "否" }}
          </template>
        </el-table-column>
        <el-table-column
          label="二级目录"
          prop="twoLevelDirectory"
        ></el-table-column>
        <el-table-column label="标签" prop="tags"></el-table-column>
        <el-table-column label="题目数量" prop="totals"></el-table-column>
        <el-table-column label="操作" width="240px">
          <template slot-scope="scope">
            <el-button type="text"> 学科分类</el-button>
            <el-button type="text">学科标签 </el-button>
            <el-button type="text" @click="edit(scope.row)">修改</el-button>
            <el-button type="text" @click="deleteSub(scope.row)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <!-- current-page当前显示的页码 -->
      <!-- page-size当前每页显示多少条数据 -->
      <div class="block" style="float: right; margin: 10px 0">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="reqParams.page"
          :page-sizes="[5, 10, 15, 20]"
          :page-size="reqParams.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
        >
        </el-pagination>
      </div>
    </el-card>
    <!-- 新增学科 -->
    <el-dialog
      title="新增学科"
      :visible.sync="dialogVisible"
      width="30%"
      @close="addSubject"
    >
      <el-form
        label-width="80px"
        :model="subject"
        :rules="subjectRules"
        ref="addSubjectRef"
        size="small"
        :inline="true"
      >
        <el-form-item label="学科名称" prop="subjectName">
          <el-input
            style="200px"
            v-model="subject.addSubjectName"
          ></el-input> </el-form-item
        ><br />
        <el-form-item label="是否显示">
          <el-switch v-model="subject.isFrontDisplay"></el-switch>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addSubject">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { list, add, remove, update } from '@/api/hmmm/subjects.js'
export default {
  name: 'subject',
  data () {
    return {
      // 请求回来的数据
      dataList: [],
      // 分页
      reqParams: {
        subjectName: '',
        page: 1, // 默认显示第一页
        pagesize: 10 // 默认一页显示10条数据
      },
      // 总的数据量
      total: 0,
      // 添加新学科的弹窗默认隐藏
      dialogVisible: false,
      // 添加弹窗
      subject: {
        addSubjectName: '',
        // 是否显示
        isFrontDisplay: 1
      },
      // 验证规则
      subjectRules: {
        subjectName: [
          {
            required: true,
            message: '请输入学科名称',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  methods: {
    // 获取所有的数据
    async getSubjectsList () {
      const { data: res } = await list(this.reqParams)
      console.log(res)
      this.dataList = res.items
      this.total = res.counts
    },
    // 清除
    clear () {
      // 清空搜索框的内容
      this.reqParams.subjectName = ''
      // 重新查询所有数据
      this.getSubjectsList()
    },
    // 删除学科
    async deleteSub (obj) {
      this.$confirm('你确认删除吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
      console.log(obj.id)
      await remove(obj.id)
      this.getSubjectsList()
    },
    // 修改学科
    edit (obj) {
      update(obj.id, obj.subjectName)
    },
    // 分页
    // 监听pagesize改变的事件
    handleSizeChange (val) {
      // 页码发生改变我们需要改变数据
      this.reqParams.pagesize = val
      this.getSubjectsList()
    },
    // 监听页码值改变的事件
    handleCurrentChange (val) {
      this.reqParams.page = val
      this.getSubjectsList()
    },
    // 新增数据
    addSubject () {
      this.$refs.addSubjectRef.validate((valid) => {
        if (!valid) return
        // 调用添加的方法
        add(this.subject)
        // 关闭对话框，隐藏数据
        this.dialogVisible = false
      })
    },
    // 监听表单关闭，重置添加按钮
    addSubject () {
      this.$refs.addSubjectRef.resetFields()
      console.log(11111)
    }
  },
  created () {
    this.getSubjectsList()
  }
}
</script>

<style lang="scss" scoped>
</style>
