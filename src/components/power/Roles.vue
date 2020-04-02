<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddRoleDialog">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template v-slot="{ row }">
            <el-row :class="['bdbottom', index1 === 0 ? 'bdtop' : '']" v-for="(item1, index1) in row.children" :key="item1.id"
            type="flex" align="middle" @close="removeRightById(row, item1.id)">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable>{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <!-- 渲染二级权限 -->
                <el-row :class="[index2 === 0 ? '' : 'bdtop']" v-for="(item2, index2) in item1.children" :key="item2.id"
                type="flex" align="middle" @close="removeRightById(row, item2.id)">
                  <el-col :span="6">
                    <el-tag type="success" closable>{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag type="warning" closable v-for="(item3) in item2.children" :key="item3.id" @close="removeRightById(row, item3.id)">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index">
        </el-table-column>
        <el-table-column prop="roleName" label="角色名称">
        </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述">
        </el-table-column>
        <el-table-column label="操作">
          <template v-slot="{ row }">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditRoleDialog(row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightsDialog(row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色的对话框 -->
    <el-dialog title="添加角色" :visible.sync="addRoleDialogVisible" width="50%" @close="addRoleDialogClosed">
      <!-- 内容主体区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑角色的对话框 -->
    <el-dialog title="编辑角色信息" :visible.sync="editRoleDialogVisible" width="50%" @close="editRoleDialogClosed">
      <!-- 内容主体区域 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightsDialogVisible"
      width="50%" @close="setRightDialogClosed">
      <!-- 内容主体区域 -->
      <el-tree :data="rightsList"
        show-checkbox node-key="id" default-expand-all
        :props="treeProps" :default-checked-keys="defKeys" ref="treeRef">
      </el-tree>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      setRightsDialogVisible: false,
      rightsList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选中的权限节点id数组
      defKeys: [],
      // 当前分配权限的角色ID
      roleId: '',
      addRoleDialogVisible: false,
      editRoleDialogVisible: false,
      // 添加角色的表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加角色的表单验证规则
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名的长度在3~16个字符之间', trigger: 'blur' }
        ]
      },
      editForm: {},
      // 编辑角色的表单验证规则
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名的长度在3~16个字符之间', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色列表数据
    async getRolesList () {
      const { data: res } = await this.$http.get('/roles')
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
    },
    // 根据ID删除权限
    async removeRightById (roleInfo, id) {
      // console.log(rolesInfo, id)
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除，则返回字符串 confirm
      // 如果用户取消删除，则返回字符串 cancel
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${roleInfo.id}/rights/${id}`)
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      // this.getRolesList()
      roleInfo.children = res.data
      this.$message.success('删除权限成功！')
    },
    // 显示分配权限对话框
    async showSetRightsDialog (role) {
      this.roleId = role.id
      // 获取所有权限
      const { data: res } = await this.$http.get('rights/tree')

      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败')
      }
      this.rightsList = res.data
      this.getDefKeys(role, this.defKeys)
      // console.log(this.defKeys)
      this.setRightsDialogVisible = true
      // console.log(role)
    },
    // 递归获得已有权限的节点id
    getDefKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getDefKeys(item, arr))
    },
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // console.log(keys.join(','))
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败!')
      }
      this.$message.success('分配权限成功!')
      this.getRolesList()
      this.setRightsDialogVisible = false
    },
    showAddRoleDialog () {
      this.addRoleDialogVisible = true
    },
    // 监听添加角色对话框的关闭事件
    addRoleDialogClosed () {
      this.$refs.addFormRef.resetFields()
      this.addForm.roleDesc = ''
    },
    // 添加角色
    addRole () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('信息验证不通过,请检查!')
        }
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败!')
        }
        this.$message.success('添加角色成功!')
        this.addRoleDialogVisible = false
        this.getRolesList()
      })
    },
    // 监听编辑角色信息对话框的关闭事件
    editRoleDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    async showEditRoleDialog (id) {
      // console.log(id)
      const { data: res } = await this.$http.get(`roles/${id}`)

      if (res.meta.status !== 200) {
        return this.$message.error('获取角色信息失败!')
      }
      this.editForm = res.data
      this.editRoleDialogVisible = true
    },
    editRoleInfo () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('信息验证不通过,请检查!')
        }
        const { data: res } = await this.$http.put(`roles/${this.editForm.roleId}`, {
          roleName: this.editForm.roleName,
          roleDesc: this.editForm.roleDesc
        })
        if (res.meta.status !== 200) {
          // console.log(res)
          return this.$message.error('更新角色信息失败!')
        }
        this.editRoleDialogVisible = false
        this.getRolesList()
        this.$message.success('更新角色信息成功!')
      })
    },
    async removeRoleById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除，则返回字符串 confirm
      // 如果用户取消删除，则返回字符串 cancel
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${id}`)
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败！')
      }
      this.getRolesList()
      this.$message.success('删除角色成功！')
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}
</style>
