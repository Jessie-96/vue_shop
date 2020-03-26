<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加分类按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <tree-table :data="cateList" :columns="columns"
      :selection-type="false" :expand-type="false"
      show-index index-text="" border
      :show-row-hover="false">
        <!-- 是否有效 -->
        <template slot="isOk" slot-scope="{ row }">
          <i class="el-icon-success" v-if="row.cat_deleted === false" style="color: #67C23A;"></i>
          <i class="el-icon-error" v-else style="color: #F56C6C"></i>
        </template>
        <!-- 排序列(分类登记列) -->
        <template slot="order" slot-scope="{ row }">
          <el-tag v-if="row.cat_level === 0" size="mini">一级</el-tag>
          <el-tag type="success" v-else-if="row.cat_level === 1" size="mini">二级</el-tag>
          <el-tag type="warning" v-else size="mini">三级</el-tag>
        </template>
        <!-- 操作列 -->
        <template slot="opt" slot-scope="{ row }">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditCateDialog(row.cat_id)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCateById(row.cat_id)">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum" :page-sizes="[5, 10, 15, 20]" :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper" :total="total">
    </el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible"
      width="50%" @close="addCateDialogClosed">
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px" >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader v-model="selectedCateId" :options="parentCateList" :props="cascaderProps"
          @change="parentCateChange" clearable>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑分类的对话框 -->
    <el-dialog title="修改分类信息" :visible.sync="editCateDialogVisible"
      width="50%" @close="editCateDialogClosed">
      <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="100px" >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      // 商品分类的数据列表
      cateList: [],
      // 请求参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      // 为talble指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          prop: 'cat_deleted',
          type: 'template',
          template: 'isOk'
        },
        {
          label: '排序',
          prop: 'cat_level',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      addCateDialogVisible: false,
      // 添加分类的表单数据
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      parentCateList: [],
      selectedCateId: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        checkStrictly: true
      },
      editCateDialogVisible: false,
      editCateForm: {},
      editCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      this.cateList = res.data.result
      this.total = res.data.total
      // console.log(this.cateList)
    },
    handleSizeChange (val) {
      // console.log(`每页 ${val} 条`)
      this.queryInfo.pagesize = val
      this.getCateList()
    },
    handleCurrentChange (val) {
      // console.log(`当前页: ${val}`)
      this.queryInfo.pagenum = val
      this.getCateList()
    },
    showAddCateDialog () {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类数据列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败！')
      }
      this.parentCateList = res.data
    },
    // 添加分类时父级选项发生变化时触发
    parentCateChange () {
      // console.log(this.selectedCateId)
      const len = this.selectedCateId.length
      if (len > 0) {
        this.addCateForm.cat_pid = this.selectedCateId[len - 1]
        this.addCateForm.cat_level = len
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 点击确定按钮，添加新的分类
    addCate () {
      // console.log(this.addCateForm)
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return this.$message.error('信息验证不通过，请检查！')
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        // console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedCateId = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    async showEditCateDialog (id) {
      const { data: res } = await this.$http.get(`categories/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类信息失败')
      }
      this.editCateForm = res.data
      this.editCateDialogVisible = true
    },
    editCateDialogClosed () {
      this.$refs.editCateFormRef.resetFields()
    },
    editCate () {
      this.$refs.editCateFormRef.validate(async valid => {
        if (!valid) return this.$message.error('信息验证不通过，请检查！')
        const { data: res } = await this.$http.put(`categories/${this.editCateForm.cat_id}`, {
          cat_name: this.editCateForm.cat_name
        })
        // console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('修改分类信息失败！')
        }
        this.editCateDialogVisible = false
        this.getCateList()
        this.$message.success('修改分类信息成功！')
      })
    },
    async removeCateById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`categories/${id}`)
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败！')
      }
      this.getCateList()
      this.$message.success('删除用户成功！')
    }
  }
}
</script>

<style lang="less" scoped>
.zk-table {
  margin-top: 15px;
}

</style>
