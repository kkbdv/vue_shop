<template>
  <div>
    <!-- // 面包屑导航区 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <!-- 搜索与添加区 -->
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getUserList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userlist" border stripe="">
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"> </el-table-column>
        <el-table-column label="邮箱" prop="email"> </el-table-column>
        <el-table-column label="手机" prop="mobile"> </el-table-column>
        <el-table-column label="角色" prop="role_name"> </el-table-column>
        <el-table-column label="状态" prop="mg_state">
          <template slot-scope="scope">
            <el-switch
              @change="userStateChange(scope.row)"
              v-model="scope.row.mg_state"
              active-color="#13ce66"
              inactive-color="#ff4949"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="small "
              @click="showEditDialog(scope.row.id)"
            ></el-button>
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="small "
              @click="removeUserById(scope.row.id)"
            ></el-button>
            <!-- 设置按钮 -->
            <el-tooltip
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="small "
                @click="showRolesDialog(scope.row)"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handlesSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主题区 -->
      <span>
        <el-form
          label-width="70px"
          :model="addForm"
          :rules="addFormRules"
          ref="addFormRef"
          status-icon
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
      </span>
      <!-- 底部区 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <!-- 表单区 -->
      <span>
        <el-form
          label-width="70px"
          :model="editForm"
          :rules="editFormRules"
          ref="editFormRef"
          status-icon
        >
          <el-form-item label="用户名">
            <el-input v-model="editForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
      </span>
      <!-- 底部 -->
      <span slot="footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配角色对话框 -->
    <el-dialog
      title="分配角色"
      :visible.sync="rolesDialogVisible"
      width="50%"
      @close="rolesClose"
    >
      <span>
        <p>当前用户名：{{ userinfo.username }}</p>
        <p>当前角色名：{{ userinfo.role_name }}</p>
        <el-select v-model="selected" placeholder="选择角色">
          <el-option
            v-for="item in options"
            :key="item.id"
            :label="item.roleName"
            :value="item.id"
          >
          </el-option>
        </el-select>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="rolesDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="rolesAdd">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    //  验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^[a-z0-9]+([._\\-]*[a-z0-9])*@([a-z0-9]+[-a-z0-9]*[a-z0-9]+.){1,63}[a-z0-9]+$/
      if (regEmail.test(value)) {
        return cb()
      } else {
        return cb(new Error('请输入合法邮箱'))
      }
    }

    //  验证手机号的规则
    var checkmobile = (rule, value, cb) => {
      const regmobile = /^1[0-9]{10}$/
      if (regmobile.test(value)) {
        return cb()
      } else {
        return cb(new Error('请输入合法手机号'))
      }
    }

    return {
      //  用户当前信息
      userinfo: {},
      options: [],
      selected: '',
      input3: '',
      queryInfo: {
        query: '',
        //  当前页码
        pagenum: 1,
        // 当前页的条数
        pagesize: 5
      },
      userlist: [],
      total: 0,
      // 控制对话框的显示与隐藏
      addDialogVisible: false,
      editDialogVisible: false,
      rolesDialogVisible: false,
      // 添加表单
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名在3-10个字符之间', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '用户名在6-15个字符之间', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkmobile }
        ]
      },
      // 查询到的用户信息对象
      editForm: {},
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkmobile }
        ]
      }
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('/users', { params: this.queryInfo })
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('数据获取失败')
      this.userlist = res.data.users
      this.total = res.data.total
    },
    // 监听pagesize改变的事件
    handlesSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听页码值得改变事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    //  用户状态改变事件
    async userStateChange (userinfo) {
      const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('更新用户状态失败!')
      }
      this.$message.success('更新状态成功!')
    },
    // 关闭对话框事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 点击添加用户
    addUser () {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        //  发起网络请求
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) this.$message.error('添加失败!')
        this.$message.success("添加成功")
        // 触发关闭对话框事件
        this.addDialogVisible = false
        //  重新获取用户列表
        this.getUserList()
      })
    },
    // 展示编辑用户的对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get(`users/${id}`)
      if (res.meta.status !== 200) return this.$message.error('查询用户失败')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 点击修改用户信息
    editUserInfo () {
      this.$refs.editFormRef.validate(async (val) => {
        if (!val) return
        const { data: res } = await this.$http.put('users/' + this.editForm.id, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        if (res.meta.status !== 200) return this.$message.error('更新失败')
        this.editDialogVisible = false
        this.getUserList()
        this.$message.success('更新成功')
      })
    },
    // 关闭编辑用户的对话框
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 删除用户
    async removeUserById (id) {
      const resConfirm = await this.$confirm('从数据库中删除该用户，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (resConfirm !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      this.getUserList()
    },
    // 打开分配角色权限按钮
    async showRolesDialog (userinfo) {
      this.userinfo = userinfo
      const { data: res } = await this.$http.get('roles')
      this.options = res.data
      this.rolesDialogVisible = true
    },
    // 提交角色
    async rolesAdd () {
      const { data: res } = await this.$http.put(`users/${this.userinfo.id}/role`, { rid: this.selected })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success(res.meta.msg)
      this.getUserList()
      this.rolesDialogVisible = false
    },
    // 关闭窗口清空填写信息
    rolesClose () {
      this.selected = ''
    }
  }
}
</script>
<style lang="less" scoped>
.el-card {
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.15) !important;
}
</style>
