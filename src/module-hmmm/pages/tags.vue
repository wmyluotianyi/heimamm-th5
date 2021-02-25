<template>
  <div class="tags-container">
    <el-card>
      <!-- 筛选 -->
      <el-row>
        <el-col :span="18">
          <el-form label-width="80px" :model="tagsModel" size="small" :inline="true">
            <el-form-item label="标签名称">
              <el-input style="200px" v-model="tagsModel.tagName"></el-input>
            </el-form-item>
            <el-form-item label="状态">
              <el-select v-model="tagsModel.state">
                <el-option :value="1" label="启用"></el-option>
                <el-option :value="0" label="禁用"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item style="text-align:right">
              <el-button @click="clear">清除</el-button>
              <el-button type="primary" @click="getTagsList">搜索</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="6" style="text-align:right">
          <el-button type="text" icon="el-icon-back" @click="$router.back()">返回学科
          </el-button>
          <el-button icon="el-icon-edit" type="success" size="small" @click="dialogVisible = true">新增标签</el-button>
        </el-col>
      </el-row>
      <!-- 数据记录 -->
      <el-alert :title="`数据一共 ${total} 条`" style="margin-bottom:15px" type="info" class="alert" :closable="false"
        show-icon></el-alert>
      <!-- 表格 -->
      <el-table :data="tagsList">
        <el-table-column label="序号" type="index" width="80px"></el-table-column>
        <el-table-column label="所属学科" prop="subjectID"></el-table-column>
        <el-table-column label="标签名称" prop="tagName"></el-table-column>
        <el-table-column label="创建者" prop="username"></el-table-column>
        <el-table-column label="创建日期">
          <template slot-scope="scope">{{ scope.row.addDate }}</template>
        </el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">{{ scope.row.state === 1 ? '已启用':'已禁用' }}</template>
        </el-table-column>
        <el-table-column label="操作" width="150px">
          <template slot-scope="scope">
            <el-button type="text" @click="triggleTags(scope.row)">{{ scope.row.state === 1 ? '禁用':'启用' }}
            </el-button>
            <el-button type="text" @click="edit(scope.row)" :disabled="scope.row.state === 1">修改</el-button>
            <el-button type="text" @click="delTags(scope.row)"
              :disabled='scope.row.state === 1 || scope.row.totals > 0'>删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页 -->
      <div class="block" style="float:right; margin:10px 0">
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
          :current-page="tagsModel.page" :page-sizes="[10,20,30,40]" :page-size="tagsModel.pageSize"
          layout="total, sizes, prev, pager, next, jumper" :total="this.total">
        </el-pagination>
      </div>
    </el-card>
    <!-- 添加标签 -->
    <el-dialog title="添加目录" :visible.sync="dialogVisible" width="30%">
      <el-form ref="form" size="small" label-width="80px" :model="tagsModel">
        <el-form-item label="所属学科" v-if="!$route.query.id" prop="subjectID">
          <el-select v-model="tagsModel.subjectID" style="width:100%">
            <el-option v-for="item in subList" :value="item.value" :label="item.label" :key="item.value">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="目录名称" prop="tagName">
          <el-input v-model="tagsModel.tagName" placeholder="请输入目录名称"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addTags">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改 -->
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
    detail,
    update
  } from '@/api/hmmm/tags'
  import {
    simple
  } from '@/api/hmmm/subjects'
  export default {
    name: 'TagsList',
    data() {
      return {
        total: 0,
        tagsModel: {
          tagName: null,
          state: null,
          page: 1,
          pageSize: 10
        },
        // table表格
        tagsList: [],
        // 添加弹窗
        dialogVisible: false,
        // 添加标签
        tagsModel: {
          subjectID: null,
          tagName: null
        },
        // 学科目录
        subList: [],
        // 是否显示修改弹窗
        show: false,
        // 修改列表
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
    methods: {
      // 获取所有数据
      async getTagsList() {
        const {
          data: res
        } = await list(this.tagsModel)
        console.log(res)
        this.tagsList = res.items
        this.total = res.counts
      },
      // 清除
      clear() {
        this.tagsModel.tagName = null
        this.tagsModel.state = null
        this.getTagsList()
      },
      // 分页
      handleSizeChange(val) {
        this.tagsModel.page = 1
        this.tagsModel.pageSize = val
        this.getTagsList()
        console.log(this.tagsModel.pageSize)
      },
      handleCurrentChange(val) {
        this.tagsModel.page = val
        this.getTagsList()
      },
      // 改变标签的状态
      triggleTags(obj) {
        changeState({
          id: obj.id,
          // 取反
          state: obj.state === 1 ? 0 : 1
        })
        // 提示用户
        this.$message.success('启动成功')
        // 重新赋值渲染
        obj.state = obj.state === 1 ? 0 : 1
      },
      // 删除
      async delTags(obj) {
        this.$confirm('删除标签吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
        await remove(obj)
        this.$message.success('删除成功')
        this.getTagsList()
      },
      // 添加
      async addTags() {
        await add(this.tagsModel)
        this.$message.success('添加成功')
      },
      // 修改
      async edit(obj) {
        this.show = true
        // 根据id获取学科和目录信息
        const result = await detail({
          id: obj.id,
          obj: obj
        })
        console.log(result);
        this.editModel = result.data
        // this.editModel.subjectID = result.data.subjectID
        // this.editModel.tagName = result.data.tagName
        // this.editModel.id = result.data.id
      },
      addEdit() {
        this.$refs.editRef.validate(async valid => {
          if (!valid) return
          await update(this.editModel)
          this.$message.success('修改成功')
          this.show = false
        })
      }
    },
    async created() {
      this.getTagsList()
      const subList = await simple()
      this.subList = subList.data
      console.log(subList);
    }
  }

</script>

<style lang="scss" scoped>
  .tags-container {
    padding: 10px;
  }

</style>
