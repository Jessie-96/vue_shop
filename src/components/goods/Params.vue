<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告信息区域 -->
      <el-alert title="注意：只允许为第三级分类设置相关参数!" type="warning" show-icon :closable="false"></el-alert>
      <!-- 商品分类选择区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类 :</span>
          <el-cascader v-model="selectedCateId" :options="cateList" :props="cascaderProps" @change="handleCateChanged"></el-cascader>
        </el-col>
      </el-row>
      <!-- 标签页区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 动态参数标签页 -->
        <el-tab-pane label="动态参数" name="many">
          <template>
            <!-- 添加参数按钮 -->
            <el-button size="mini" type="primary" @click="showAddParamDialog" :disabled="isDisabled">添加参数</el-button>
            <!-- 参数信息列表 -->
            <el-table :data="manyTableData" border stripe>
              <el-table-column type="expand">
                <template v-slot="{ row }">
                  <el-row type="flex" align="middle">
                    <el-col>
                      <el-tag v-for="(tag, index) in row.attr_vals" :key="index" closable type="primary" @close="removeTag(index, row)">{{tag}}</el-tag>
                      <el-input class="input-new-tag" v-if="row.inputVisible"
                        v-model="row.inputValue" ref="saveTagInput" size="small"
                        @keyup.enter.native="handleInputConfirm(row)" @blur="handleInputConfirm(row)">
                      </el-input>
                      <el-button v-else class="button-new-tag" size="small" @click="showInput(row)">+ New Tag</el-button>
                    </el-col>
                  </el-row>
                </template>
              </el-table-column>
              <el-table-column type="index"></el-table-column>
              <el-table-column prop="attr_name" label="参数名称"></el-table-column>
              <el-table-column label="操作">
                <template v-slot="{ row }">
                  <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEdidParamDialog(row.attr_id)">编辑</el-button>
                  <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParamById(row.attr_id)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </template>
        </el-tab-pane>
        <!-- 静态参数标签页 -->
        <el-tab-pane label="静态属性" name="only">
          <template>
            <!-- 添加属性按钮 -->
            <el-button size="mini" type="primary" :disabled="isDisabled" @click="showAddParamDialog">添加属性</el-button>
            <!-- 参数信息列表 -->
            <el-table :data="onlyTableData" border stripe>
              <el-table-column type="expand">
                <template v-slot="{ row }">
                  <el-row type="flex" align="middle">
                    <el-col>
                      <el-tag v-for="(tag, index) in row.attr_vals" :key="index" closable type="primary" @close="removeTag(index, row)">{{tag}}</el-tag>
                      <el-input class="input-new-tag" v-if="row.inputVisible"
                        v-model="row.inputValue" ref="saveTagInput" size="small"
                        @keyup.enter.native="handleInputConfirm(row)" @blur="handleInputConfirm(row)">
                      </el-input>
                      <el-button v-else class="button-new-tag" size="small" @click="showInput(row)">+ New Tag</el-button>
                    </el-col>
                  </el-row>
                </template>
              </el-table-column>
              <el-table-column type="index"></el-table-column>
              <el-table-column prop="attr_name" label="参数名称"></el-table-column>
              <el-table-column label="操作">
                <template v-slot="{ row }">
                  <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEdidParamDialog(row.attr_id)">编辑</el-button>
                  <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParamById(row.attr_id)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </template>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数/属性对话框 -->
    <el-dialog :title="'添加' + dialogTitle" :visible.sync="addParamDialogVisible"
      width="50%" @close="addParamDialogClosed">
      <el-form :model="addParamForm" :rules="addParamFormRules" ref="addParamFormRef" label-width="100px" >
        <el-form-item :label="dialogTitle" prop="attr_name">
          <el-input v-model="addParamForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addParamDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParam">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑参数/属性对话框 -->
    <el-dialog :title="'编辑' + dialogTitle" :visible.sync="editParamDialogVisible"
      width="50%" @close="editParamDialogClosed">
      <el-form :model="editParamForm" :rules="editParamFormRules" ref="editParamFormRef" label-width="100px" >
        <el-form-item :label="dialogTitle" prop="attr_name">
          <el-input v-model="editParamForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editParamDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParam">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      cateList: [],
      selectedCateId: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      addParamDialogVisible: false,
      addParamForm: {
        attr_name: ''
      },
      addParamFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      editParamDialogVisible: false,
      editParamForm: {},
      editParamFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 3 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类信息失败')
      }
      this.cateList = res.data
    },
    handleCateChanged () {
      // console.log(this.selectedCateId)
      if (this.selectedCateId.length !== 3) {
        this.selectedCateId = []
        this.manyTableData = []
        this.onlyTableData = []
        return false
      }
      this.getParamsList()
    },
    handleTabClick () {
      // console.log(this.activeName)
      this.getParamsList()
    },
    async getParamsList () {
      if (this.selectedCateId.length !== 3) {
        this.selectedCateId = []
        return false
      }
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName }
        })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败')
      }
      // console.log(res)
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    showInput (row) {
      row.inputVisible = true
      // $nextTick 方法的作用，就是当页面上的元素被重新渲染之后，才会执行回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    async handleInputConfirm (row) {
      // console.log(row)
      if (!row.inputValue.trim()) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      this.saveAttrValues(row)
      row.inputVisible = false
      row.inputValue = ''
    },
    async saveAttrValues (row) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数项失败！')
      }
      this.$message.success('修改参数项成功！')
    },
    // 删除对应的参数可选项
    removeTag (index, row) {
      row.attr_vals.splice(index, 1)
      this.saveAttrValues(row)
    },
    showAddParamDialog () {
      this.addParamDialogVisible = true
    },
    addParamDialogClosed () {
      this.$refs.addParamFormRef.resetFields()
    },
    addParam () {
      this.$refs.addParamFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('信息验证不通过，请检查')
        }
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addParamForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败！')
        }
        this.getParamsList()
        this.addParamDialogVisible = false
        this.$message.success('添加参数成功！')
      })
    },
    async showEdidParamDialog (id) {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数信息失败！')
      }
      this.editParamForm = res.data
      this.editParamDialogVisible = true
    },
    editParamDialogClosed () {
      this.$refs.editParamFormRef.resetFields()
    },
    editParam () {
      this.$refs.editParamFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('信息验证不通过，请检查')
        }
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editParamForm.attr_id}`, {
          attr_name: this.editParamForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数信息失败！')
        }
        this.getParamsList()
        this.editParamDialogVisible = false
        this.$message.success('修改参数信息成功！')
      })
    },
    async removeParamById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(` categories/${this.cateId}/attributes/${id}`)
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败！')
      }
      this.getParamsList()
      this.$message.success('删除参数成功！')
    }
  },
  computed: {
    isDisabled () {
      if (this.selectedCateId.length !== 3) {
        return true
      }
      return false
    },
    cateId () {
      if (this.selectedCateId.length === 3) {
        return this.selectedCateId[2]
      }
      return null
    },
    dialogTitle () {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-cascader {
  margin-left: 10px;
}
.el-tag {
  margin-right: 10px;
  margin-bottom: 5px;
}
// .button-new-tag {
//   margin-left: 10px;
//   height: 32px;
//   line-height: 30px;
//   padding-top: 0;
//   padding-bottom: 0;
// }
.input-new-tag {
  width: 90px;
  // margin-left: 10px;
  // vertical-align: bottom;
}
</style>
