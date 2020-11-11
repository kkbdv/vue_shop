<template>
<div>
  <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item><a href="/">商品管理</a></el-breadcrumb-item>
      <el-breadcrumb-item><a href="/">参数列表</a></el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片 -->
    <el-card>
      <el-alert
      show-icon
      :closable="false"
      title="只允许为第三级分类设置相关参数！"
      type="warning">
      </el-alert>
      <!-- 级联选择框 -->
      <div class="choose">
        选择商品分类
        <el-cascader
          :props="cateProps"
          expand-trigger="hover"
          :options="cateOptions"
          v-model="selectedOptions"
          @change="handleCatehange">
        </el-cascader>
      </div>
      <!-- 选项卡 -->
       <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisable" @click="openAddDialog">添加参数</el-button>
          <!-- 表格 -->
          <el-table
          :data="ManyDataTable"
          style="width: 100%"
          border stripe>
          <el-table-column
            type="expand">
            <!-- 循环渲染tag标签 -->
           <template slot-scope="scope">
             <el-tag closable @close='tagClose(index,scope.row)' class="tag" v-for="(item,index) in scope.row.attr_vals" :key="index">{{item}}</el-tag>
             <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
           </template>
          </el-table-column>
          <el-table-column
            type="index">
          </el-table-column>
          <el-table-column
            prop="attr_name"
            label="参数名称"
           >
          </el-table-column>
          <el-table-column
            label="操作"
           >
           <template slot-scope="scope">
             <el-button size="mini" type="primary" icon="el-icon-edit" @click="editDialogOpen(scope.row)">修改</el-button>
             <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteAttribute(scope.row)">删除</el-button>
           </template>
          </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
           <el-button type="primary" size="mini" :disabled="isBtnDisable" @click="openAddDialog">添加属性</el-button>
           <!-- 表格 -->
          <el-table
          :data="OnlyDataTable"
          style="width: 100%"
          border stripe>
          <el-table-column
            type="expand">
            <el-table-column
            type="expand">
            <!-- 循环渲染tag标签 -->
           <template slot-scope="scope">
             <el-tag closable @close='tagClose(index,scope.row)' class="tag" v-for="(item,index) in scope.row.attr_vals" :key="index">{{item}}</el-tag>
             <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
           </template>
          </el-table-column>
          </el-table-column>
          <el-table-column
            type="index">
          </el-table-column>
          <el-table-column
            prop="attr_name"
            label="静态参数"
           >
          </el-table-column>
          <el-table-column
            label="操作"
           >
           <template slot-scope="scope">
             <el-button size="mini" type="primary" icon="el-icon-edit" @click="editDialogOpen(scope.row)">修改</el-button>
             <el-button size="mini" type="danger" icon="el-icon-delete" @click="deleteAttribute(scope.row)" >删除</el-button>
           </template>
          </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数对话框 -->
    <el-dialog
      :title="'添加'+addTitle"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClose"
      >
      <span>
        <!-- 表单 -->
        <el-form ref="addFromRef" :model="addForm" label-width="80px" :rules="addRules">
          <el-form-item :label="addLabel" prop="attr_name">
            <el-input v-model="addForm.attr_name"></el-input>
          </el-form-item>
        </el-form>

      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addAttribute">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数对话框 -->
    <el-dialog
      :title="'修改'+editTitle"
      :visible.sync="editDialogVisible"
      width="50%"
      @close = "editDialogClose"
      >
      <span>
        <!-- 表单 -->
        <el-form ref="editFromRef" :model="editForm" label-width="80px" :rules="editRules">
          <el-form-item :label="editLabel" prop="attr_name">
            <el-input v-model="editForm.attr_name"></el-input>
          </el-form-item>
        </el-form>

      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editAttribute">确 定</el-button>
      </span>
    </el-dialog>
</div>
</template>
<script>
export default {
  data () {
    return {
      cateOptions: [],
      selectedOptions: [],
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      addDialogVisible: false,
      editDialogVisible: false,
      // 激活的选项卡
      activeName: 'many',
      // 动态对象
      ManyDataTable: [],
      // 静态对象
      OnlyDataTable: [],
      // 添加表单对象
      addForm: {
        attr_name: ''
      },
      editForm: {
        attr_name: ''
      },
      // 校验表单对象
      addRules: {
        attr_name: [
          { required: true, message: '输入内容不能为空', trigger: 'blur' }
        ]
      },
      editRules: {
        attr_name: [
          { required: true, message: '输入内容不能为空', trigger: 'blur' }
        ]
      }

    }
  },
  computed: {
    isBtnDisable () {
      if (this.selectedOptions.length !== 3) return true
      return false
    },
    // 计算出当前选中的三级分类的ID
    cateId () {
      if (this.selectedOptions.length === 3) {
        return this.selectedOptions[2]
      }
      return null
    },
    // 动态计算 对话框 文本
    addTitle () {
      if (this.activeName === 'many') return '动态参数'
      return '静态属性'
    },
    // 动态计算 文本
    editTitle () {
      if (this.activeName === 'many') return '动态参数'
      return '静态属性'
    },
    // 计算文本框
    addLabel () {
      if (this.activeName === 'many') return '动态参数'
      return '静态属性'
    },
    editLabel () {
      if (this.activeName === 'many') return '动态参数'
      return '静态属性'
    }

  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.cateOptions = res.data
    },
    // 触发获取数据
    handleCatehange (selected) {
      this.getCateAtrribute()
    },
    handleTabClick () {
      this.getCateAtrribute()
    },
    // 根据所选id和所处面板,获取对应参数 ，复用方法
    async getCateAtrribute () {
      if (this.selectedOptions.length !== 3) {
        this.selectedOptions = []
        this.ManyDataTable = []
        this.OnlyDataTable = []
        return
      }
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) this.$message.error(res.meta.msg)
      res.data.forEach(item => {
        // 先判断是否为空，避免出现字符串分割为空的情况
        item.attr_vals = item.attr_vals ? item.attr_vals.split(" ") : []
        // 控制文本框的显示与隐藏
        item.inputVisible = false
        // 文本框中输入的值
        item.inputValue = ''
      })
      console.log(res.data)

      if (this.activeName === 'only') this.OnlyDataTable = res.data
      this.ManyDataTable = res.data
    },
    // 添加事件
    addAttribute () {
      this.$refs.addFromRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, { attr_name: this.addForm.attr_name, attr_sel: this.activeName })
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
        this.$message.success(res.meta.msg)
        this.addDialogVisible = false
        this.getCateAtrribute()
      })
    },
    // 修改事件
    editAttribute () {
      this.$refs.editFromRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.editForm.cat_id}/attributes/${this.editForm.attr_id}`, { attr_name: this.editForm.attr_name, attr_sel: this.activeName })
        if (res.meta.status !== 200) return this.$message.error(this.meta.msg)
        this.$message.success("修改成功")
        this.getCateAtrribute()
        this.editDialogVisible = false
      })
    },
    // 关闭清空事件
    addDialogClose () {
      this.$refs.addFromRef.resetFields()
    },
    // 打开窗口
    openAddDialog () {
      this.addDialogVisible = true
    },
    // 给对象传递这一行的值
    editDialogOpen (e) {
      this.editForm = e
      this.editDialogVisible = true
    },
    editDialogClose () {
      this.$refs.editFromRef.resetFields()
    },
    // 删除参数
    async deleteAttribute (e) {
      const Confirmres = await this.$confirm('是否继续删除?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (Confirmres !== 'confirm') return
      const { data: res } = await this.$http.delete(`categories/${e.cat_id}/attributes/${e.attr_id}`)
      if (res.meta.status !== 200) return this.$message.error("操作失败")
      this.$message.success("删除成功")
      this.getCateAtrribute()
    },
    // 点击切换输入事件
    showInput (e) {
      e.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 输入和失去焦点事件
    handleInputConfirm (e) {
      if (e.inputValue.trim().length === 0) {
        e.inputValue = ''
        e.inputVisible = false
      }
      // 后续处理正确的输入
      e.attr_vals.push(e.inputValue.trim())
      e.inputValue = ''
      e.inputVisible = false
      // 需要发起请求
      this.saveAttribute(e)
    },
    // 保存数组操作
    async saveAttribute (e) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${e.attr_id}`, {
        attr_name: e.attr_name, attr_sel: this.activeName, attr_vals: e.attr_vals.join(" ")
      })
      if (res.meta.status !== 200) this.$message.error('操作失败')
      this.$message.success("操作成功")
    },
    // 关闭标签触发跟新
    async tagClose (i, e) {
      // 分开数组，从i开始，分一个
      e.attr_vals.splice(i, 1)
      this.saveAttribute(e)
    }
  }
}
</script>
<style lang="less" scoped>
.choose{
  margin-top: 15px;
}
.tag{
  margin: 0 5px;
}
.input-new-tag{
  width: 120px;
}
</style>
