<template>
  <div class="directorys-container">
    <el-card>
      <!-- 筛选 -->
      <el-row>
        <el-col :span="18">
          <el-form label-width="80px" :model="reqParams" size="small" :inline="true">
            <el-form-item label="目录名称">
              <el-input style="200px" v-model="reqParams.directoryName"></el-input>
            </el-form-item>
            <el-form-item label="状态">
              <el-select v-model="reqParams.state">
                <el-option :value="1" label="启用"></el-option>
                <el-option :value="0" label="禁用"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item style="text-align:right">
              <el-button @click="clear">清除</el-button>
              <el-button type="primary" @click="getList">搜索</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="6" style="text-align:right">
          <el-button type="text" icon="el-icon-back" @click="$router.back()">返回学科
          </el-button>
          <el-button icon="el-icon-edit" type="success" size="small" @click="dialogVisible = true">新增目录</el-button>
        </el-col>
      </el-row>
      <!-- 数据记录 -->
      <el-alert :title="`数据一共 ${total} 条`" style="margin-bottom:15px" type="info" class="alert" :closable="false"
        show-icon></el-alert>
      <!-- 表格 -->
      <el-table :data="dirList">
        <el-table-column label="序号" type="index" width="80px"></el-table-column>
        <el-table-column label="所属学科" prop="subjectName"></el-table-column>
        <el-table-column label="目录名称" prop="directoryName"></el-table-column>
        <el-table-column label="创建者" prop="username"></el-table-column>
        <el-table-column label="创建日期" prop="addDate">
          <template slot-scope="scope">{{ scope.row.addDate }}</template>
        </el-table-column>
        <el-table-column label="面试题数量" prop="totals"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">{{scope.row.state===1?'已启用':'已禁用'}}</template>
        </el-table-column>
        <el-table-column label="操作" width="150px">
          <template slot-scope="scope">
            <el-button type="text" @click="triggle(scope.row)">{{scope.row.state===1?'禁用':'启用'}}</el-button>
            <!-- 如果是禁用状态就禁止修改 -->
            <el-button type="text" @click="edit(scope.row)" :disabled="scope.row.state===1">修改</el-button>
            <!-- 不是禁用状态数据大于0 的时候才可以修改 -->
            <el-button type="text" @click="delDir(scope.row)" :disabled="scope.row.state===1||scope.row.totals > 0">删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <!-- <div class="block">
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
          :current-page="reqParams.page" :page-sizes="[5,10,15,20]" :page-size="reqParams.pageSize"
          layout="total, sizes, prev, pager, next, jumper" :total="total">
        </el-pagination>
      </div> -->
      <div class="block" style="float:right; margin:10px 0">
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
          :current-page="reqParams.page" :page-sizes="[5, 10, 15, 20]" :page-size="reqParams.pagesize"
          layout="total, sizes, prev, pager, next, jumper" :total="total">
        </el-pagination>
      </div>
    </el-card>
    <!-- 新增弹窗 -->
    <el-dialog title="提示" :visible.sync="dialogVisible" width="30%">
      <el-form :model="addDirModel" :rules="addDirRules" ref="addDirRef" :inline="true">
        <el-form-item label="所属学科">
          <el-input style="200px" v-model="addDirModel.subjectID"></el-input>
        </el-form-item>
        <el-form-item label="目录名称" prop="directoryName">
          <el-input style="200px" v-model="addDirModel.directoryName"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addDir">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改目录弹窗 -->
    <el-dialog title="修改目录" :visible.sync="show" width="30%">
      <el-form ref="editRef" size="small" label-width="80px" :rules="editRules" :model="editModel">
        <el-form-item label="所属学科" v-if="!$route.query.id" prop="subjectID">
          <el-select v-model="editModel.subjectID" style="width:100%">
            <el-option v-for="item in subList" :value="item.value" :label="item.label" :key="item.value">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="标签名称" prop="tagName">
          <el-input v-model="editModel.tagName" placeholder="请输入目录名称"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="show = false">取 消</el-button>
        <el-button type="primary" @click="addEdit">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  import {
    list,
    changeState,
    remove,
    add,
    detail
  } from '@/api/hmmm/directorys'
  import {
    simple
  } from '@/api/hmmm/subjects'
  export default {
    name: 'DirectorysList',
    data() {
      return {
        reqParams: {
          directoryName: null,
          state: null,
          page: 1,
          pageSize: 10
        },
        total: 0,
        dirList: [],
        dialogVisible: false,
        addDirModel: {
          subjectID: 'java',
          directoryName: null
        },
        // 验证规则
        addDirRules: {
          directoryName: [{
            request: true,
            message: '请输入目录名称',
            triggle: 'blur'
          }]
        },
        // 编辑弹窗
        show: false,
        // 学科目录
        subList: [],
        // 编辑
        editModel: {
          id: null,
          subjectID: null,
          tagName: null
        },
        // 修改验证
        editRules: {
          subjectID: [{
            type: 'number',
            message: '请选择所属学科',
            tirgger: 'blur'
          }],
          tagName: [{
            required: true,
            message: '请输入标签名称',
            tirgger: 'blur'
          }]
        },
      }
    },
    async created() {
      this.getList()
      const subList = await simple()
      this.subList = subList.data
      console.log(subList);
    },
    methods: {
      // 查询列表
      async getList() {
        const {
          data: res
        } = await list(this.reqParams)
        // console.log(res)
        this.dirList = res.items
        this.total = res.counts
      },
      // 长度发生变化
      handleSizeChange(val) {
        console.log(val)
        this.reqParams.pageSize = val
        this.getList()
      },
      // 页码发生改变
      handleCurrentChange(val) {
        this.reqParams.page = val
        this.getList()
      },
      // 清空
      clear() {
        this.reqParams.directoryName = ''
        this.reqParams.state = null
        this.getList()
      },
      // 新增
      addDir() {
        this.$refs.addDirRef.validate(async valid => {
          if (!valid) return
          console.log(this.addDirModel)
          await add(this.addDirModel)
          this.$message.success("添加成功")
          this.getList()
          this.dialogVisible = false
        })
      },
      // 修改
      async edit(obj) {
        this.show = true
        const detailRes = await detail({
          id: obj.id,
          obj: obj
        })
        this.editModel = detailRes.data
      },
      // 确认修改
      addEdit() {
        this.$refs.editRef.validate(async valid => {
          if (!valid) return
          await update(this.editModel)
          this.$message.success('修改成功')
          this.show = false
        })
      },
      // 切换禁用的状态,传入id和状态
      triggle(obj) {
        changeState({
          id: obj.id,
          state: obj.state === 1 ? 0 : 1
        })
        this.$message.success('修改成功')
        obj.state = obj.state === 1 ? 0 : 1
      },
      // 删除
      async delDir(obj) {
        await this.$confirm('你确定删除吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        await remove(obj)
        this.$message.success('删除成功')
        this.getList()
      },
    }
  }

</script>

<style lang="scss" scoped>
  .directorys-container {
    padding: 10px;
  }

</style>
