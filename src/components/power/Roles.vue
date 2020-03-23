<template>
  <div>
  <!--面包屑导航区域-->
  <el-breadcrumb separator-class="el-icon-arrow-right">
    <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
    <el-breadcrumb-item>角色管理</el-breadcrumb-item>
    <el-breadcrumb-item>角色列表</el-breadcrumb-item>
  </el-breadcrumb>
  <!--卡片视图-->
  <el-card>
    <el-row>
      <el-col>
        <el-button type="primary" @click="showAddRoleDialog">添加角色</el-button>
      </el-col>
    </el-row>
    <!-- 角色列表区域 -->
    <el-table :data="rolesList" border stripe>
      <!-- 展开列 -->
      <el-table-column type="expand">
      <template slot-scope="scope">
        <el-row v-for="(item1,i1) in scope.row.children" :key="item1.id" :class="['bdbottom',i1 === 0 ? 'bdtop': '','vcenter']" >
          <!-- 渲染一级权限 -->
          <el-col :span="5" >
            <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
            <i class="el-icon-caret-right"></i>
          </el-col>
          <!-- 渲染二级和三级权限 -->
          <el-col :span="19">
            <!-- 通过 for 循环 嵌套渲染二级权限-->
            <el-row v-for="(item2,i2) in item1.children" :key="item2.id" :class="[i2 ===0 ?'':'bdtop','vcenter']">
              <el-col :span="6">
                <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="18">
                <el-tag  v-for="(item3) in item2.children" :key="item3.id" type="warning" closable @close="removeRightById(scope.row,item3.id)">
                {{item3.authName}}
                </el-tag>
              </el-col>
            </el-row>
          </el-col>
        </el-row>

      </template>
      </el-table-column>
      <el-table-column type="index" label="#"></el-table-column>
      <el-table-column  label="角色名称" prop="roleName"></el-table-column>
      <el-table-column  label="角色描述" prop="roleDesc"></el-table-column>

      <el-table-column  label="操作" width="300px">
        <template slot-scope="scope">
        <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditRoleDialog(scope.row.id)">编辑</el-button>
        <el-button type="danger" size="mini" icon="el-icon-delete" @click="deleteRole(scope.row.id)">删除</el-button>
        <el-button type="warning" size="mini" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
        </template>
      </el-table-column>
    </el-table>
  </el-card>
    <!-- 添加角色的对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="setAddRoleDialogVisible"
      width="30%" @close="addRoleFormClosed">
      <el-form :model="addRoleForm" :rules="addRoleRules" label-width="100px" ref="addRoleFormRef">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="setAddRoleDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addRole()">确 定</el-button>
  </span>
    </el-dialog>
    <!--修改角色的对话框-->
    <el-dialog
      title="修改角色信息"
      :visible.sync="editRoleFormDialogVisible"
      width="30%"
      @close="editRoleFormDialogClosed">
      <el-form :model="editRoleForm" :rules="editRoleFormRules" ref="editRoleFormRef" label-width="70px" >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="editRoleFormDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editRoleInfo">确 定</el-button>
  </span>
    </el-dialog>
    <!--分配权限的对话框-->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%" @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  name: 'Roles',
  data() {
    return {
      roleId: '',
      rolesList: [],
      // 控制分配权限对话框的显示于隐藏
      setRightDialogVisible: false,
      rightsList: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认的节点Id数值
      defKeys: [],
      // 添加角色对话框的显示与隐藏
      setAddRoleDialogVisible: false,
      // 添加角色表单
      addRoleForm: {},
      // 添加角色规则
      addRoleRules: {
        // 验证角色名称是否合法
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        // 验证角色描述是否合法
        roleDesc: [

        ]
      },
      // 编辑角色对话框的显示与隐藏
      editRoleFormDialogVisible: false,
      // 编辑角色表单
      editRoleForm: [],
      // 编辑角色表单规则
      editRoleFormRules: {
        // 验证角色名称是否合法
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        // 验证角色描述是否合法
        roleDesc: [
        ]
      }
    }
  },
  methods: {
    // 获得所有角色的列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败!')
      this.rolesList = res.data
    },
    // 根据id删除对应的权限
    async removeRightById(role, rightId) {
      // 弹框提示用户是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除!')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败!')

      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限数据失败!')

      // 把获取到的权限数据保存到data中
      this.rightsList = res.data
      this.getLeafKeys(role, this.defKeys)

      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限id，并保存到defKeys 数组中
    getLeafKeys(node, arr) {
      // 如果当前 node 节点不包含children 属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 分配权限事件
    async allotRights(roleId) {
      const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) return this.$message.error('分配权限失败! ')

      this.$message.success('分配权限成功! ')
      this.getRolesList()
      this.setRightDialogVisible = false
    },
    // 显示添加角色对话框
    showAddRoleDialog() {
      this.setAddRoleDialogVisible = true
    },
    // 添加角色事件
    addRole() {
      this.$refs.addRoleFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRoleForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败! ')
        }
        this.$message.success('添加角色成功!')
        this.setAddRoleDialogVisible = false
        this.getRolesList()
      })
    },
    // 关闭添加角色对话框事件
    addRoleFormClosed() {
      this.addRoleForm = {}
    },
    //
    async deleteRole(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 删除角色
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除角色失败! ')
      this.$message.success('删除角色成功! ')
      this.getRolesList()
    },
    // 编辑角色对话框的显示与隐藏
    async showEditRoleDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取角色信息失败! ')
      this.editRoleForm = res.data
      this.editRoleFormDialogVisible = true
    },
    // 监听修改角色对话框的关闭事件
    editRoleFormDialogClosed() {
      this.$refs.editRoleFormRef.resetFields()
    },
    // 提交修改角色信息
    editRoleInfo() {
      this.$refs.editRoleFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('roles/' + this.editRoleForm.roleId, {
          roleName: this.editRoleForm.roleName,
          roleDesc: this.editRoleForm.roleDesc
        })
        if (res.meta.status !== 200) return this.$message.error('修改角色信息失败! ')
        // 关闭对话框
        this.editRoleFormDialogVisible = false
        // 刷新数据列表
        this.getRolesList()
        // 提示修改成功
        this.$message.success('更新角色信息成功! ')
      })
    }
  },
  created() {
    this.getRolesList()
  }

}
</script>

<style lang="less" scoped>
  .el-tag{
    margin: 7px;
  }
  .bdtop{
      border-top: 1px solid #eee;
  }
  .bdbottom{
      border-bottom: 1px solid #eee;
  }
  .vcenter {
    display: flex;
    align-items: center;
  }
</style>
