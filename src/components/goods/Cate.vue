<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图-->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table :data="catelist"
                  :columns="columns"
                  :selection-type="false"
                  :expand-type="false"
                  show-index
                  index-text="#"
                  border
                  :show-row-hover="false"
                  class="treeTable">
        <!-- 是否有效-->
        <template slot-scope="scope" slot="isOk">
          <i class="el-icon-success" v-if="scope.row.deleted === 0" style="color: lightgreen"></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.level === 0">一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.level === 1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditCatalogDialog(scope.row.id)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteCatalog(scope.row.id)">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="30%" @close="addCateDialogClosed">
      <el-form ref="addCateFormRef" :model="addCateForm" :rules="addCateFormRules" label-width="100px">
        <el-form-item label="分类名称:" prop="name">
          <el-input v-model="addCateForm.name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类: ">
          <!-- options 用来指定数据源 -->
          <!-- props 用来指定配置对象-->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="addCateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
    </el-dialog>
    <!-- 编辑分类对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="editCatalogFormDialogVisible"
      width="30%" @close="editCatalogDialogClosed">
      <el-form :model="editCatalogForm" :rules="editCatalogFormRules" ref="editCatalogFormRef">
        <el-form-item label="分类名称" prop="name">
          <el-input v-model="editCatalogForm.name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="editCatalogFormDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editCatalogInfo">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'Cate',
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表， 默认为空
      catelist: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [{
        label: '分类名称',
        prop: 'name'
      }, {
        label: '是否有效',
        // 表示将当前列定义为模板列
        type: 'template',
        // 表示当前这一列使用模板名称
        template: 'isOk'
      }, {
        label: '排序',
        // 表示将当前列定义为模板列
        type: 'template',
        // 表示当前这一列使用模板名称
        template: 'order'
      }, {
        label: '操作',
        // 表示将当前列定义为模板列
        type: 'template',
        // 表示当前这一列使用模板名称
        template: 'opt'
      }],
      addCateForm: {
        // 将要添加的分类名称
        name: '',
        // 父级分类Id
        pid: 0,
        // 分类层级,默认为一级分类
        level: 0

      },
      // 添加分类表单的验证规则对象
      addCateFormRules: {
        name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        checkStrictly: true,
        expandTrigger: 'hover',
        value: 'id',
        label: 'name',
        children: 'children'
      },
      // 选中的父级分类的Id数组
      selectedKeys: [],
      // 编辑分类对象
      editCatalogForm: [],
      // 编辑分类对话框的显示与隐藏
      editCatalogFormDialogVisible: false,
      // 编辑分类的规则
      editCatalogFormRules: {
        // 验证分类名称是否合法
        name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 处理商品分类数据
    isDelete(keyCategories, cat) {
      if (cat.pid === 0) {
        return (cat.deleted === 1)
      } else if (cat.deleted === 1) {
        return true
      } else {
        const parentCat = keyCategories[cat.pid]
        if (!parentCat) return true
        return this.isDelete(keyCategories, parentCat)
      }
    },
    getTreeResult(keyCategories, categories, type) {
      const result = []
      for (const idx in categories) {
        const cat = categories[idx]
        // 判断是否被删除
        if (this.isDelete(keyCategories, cat)) continue
        if (cat.pid === 0) {
          result.push(cat)
        } else {
          if (cat.level >= type) continue
          const parentCat = keyCategories[cat.pid]
          if (!parentCat) continue
          if (!parentCat.children) {
            parentCat.children = []
          }
          parentCat.children.push(cat)
        }
      }
      return result
    },
    dealCatalog(categories, conditions, type) {
      const keyCategories = _.keyBy(categories, 'id')
      const result = this.getTreeResult(keyCategories, categories, type)
      if (conditions) {
        const count = result.length
        const pagesize = parseInt(conditions.pagesize)
        const pagenum = parseInt(conditions.pagenum)
        const result2 = _.take(_.drop(result, (pagenum - 1) * pagesize), pagesize)
        const resultDta = {}
        resultDta.total = count
        resultDta.pagenum = pagenum
        resultDta.pagesize = pagesize
        resultDta.result = result2
        return resultDta
      }
      return result
    },
    // 获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('manage/catalogs', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取商品分类数据列表失败! ')
      const result = this.dealCatalog(res.data, this.queryInfo, 3)
      this.catelist = result.result
      this.total = result.total
    },
    // 监听 pageSize 改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮展示添加分类的对话框
    showAddCateDialog() {
      // 先获取父级分类
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('manage/catalogs', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取商品分类数据列表失败! ')
      this.parentCateList = this.dealCatalog(res.data, null, 2)
    },
    // 选中项发生变化触发这个函数
    parentCateChanged() {
      // 如果 selectedKeys 数组中的length 大于0， 证明 选中的父级分类
      // 反之 就说明没有选中任何父级分类
      console.log(this.selectedKeys)
      if (this.selectedKeys.length > 0) {
        // 父级分类id
        this.addCateForm.pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 为当前分类的等级赋值
        this.addCateForm.level = this.selectedKeys.length
      } else {
        // 父级分类Id
        this.addCateForm.pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.level = 0
      }
    },
    // 点击按钮添加新的分类
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('manage/catalogs', this.addCateForm)

        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败! ')
        }
        this.$message.success('添加分类成功! ')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听对话量的关闭事件，重置表单数据
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.level = 0
      this.addCateForm.pid = 0
    },
    // 删除分类
    async deleteCatalog(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('manage/catalogs/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除失败分类! ')
      this.$message.success('删除分类成功! ')
      // console.log(this.catelist)
      this.getCateList()
    },
    // 显示编辑分类对话框
    async showEditCatalogDialog(id) {
      const { data: res } = await this.$http.get('manage/catalogs/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取分类信息失败! ')
      this.editCatalogForm = res.data
      this.editCatalogFormDialogVisible = true
    },
    // 提交更新分类
    async editCatalogInfo() {
      const { data: res } = await this.$http.put('manage/catalogs/' + this.editCatalogForm.id, this.editCatalogForm)
      if (res.meta.status !== 200) return this.$message.error('修改分类失败! ')
      this.editCatalogFormDialogVisible = false
      this.getCateList()
      this.$message.success('修改分类成功! ')
    },
    // 监听修改角色对话框的关闭事件
    editCatalogDialogClosed() {
      this.$refs.editCatalogFormRef.resetFields()
    }
  }

}
</script>

<style lang="less" scoped>
  .treeTable {
    margin-top: 15px;
  }

  .el-cascader {
    width: 100%
  }
</style>
