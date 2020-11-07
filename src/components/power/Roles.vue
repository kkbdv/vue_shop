<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="roleSearch"
            class="input-with-select"
          >
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="showAddDialog">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区 -->
      <el-table ref="rolesTable" :data="rolesList" border stripe>
        <!-- 索引列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdtop', i1 === 0 ? '' : 'bdbottom', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>

                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-col>
                      <el-tag
                        type="warning"
                        :key="item3.id"
                        v-for="item3 in item2.children"
                        closable
                        @close="removeRightById(scope.row, item3.id)"
                        >{{ item2.authName }}</el-tag
                      ></el-col
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"> </el-table-column>
        <el-table-column prop="roleName" label="角色名称"> </el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"> </el-table-column>
        <el-table-column prop="roleDesc" label="操作">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showEditRole(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="deleteRole(scope.row.id)"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 修改角色对话框 -->
    <el-dialog title="修改角色" :visible.sync="editDialog" width="50%">
      <span>
        <el-form
          label-width="80px"
          :model="editRoleForm"
          ref="editRoleFormRef"
          :rules="editRoleRules"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editRoleForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述">
            <el-input v-model="editRoleForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialog = false">取 消</el-button>
        <el-button type="primary" @click="editFormConfirm">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addDialog"
      width="50%"
      @close="addDialogClosed"
    >
      <span>
        <el-form
          label-width="80px"
          :model="addRoleForm"
          ref="addRoleFormRef"
          :rules="addRoleRules"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addRoleForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addRoleForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialog = false">取 消</el-button>
        <el-button type="primary" @click="addFormConfirm">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限的对话框 -->
    <el-dialog
      title="权限分配"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClose"
    >
      <span
        ><el-tree
          :data="rightsList"
          :props="treeProps"
          :default-checked-keys="defkeys"
          show-checkbox
          node-key="id"
          default-expand-all
          ref="rightsRef"
        ></el-tree
      ></span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      rolesList: [],
      rightsList: [],
      editDialog: false,
      addDialog: false,
      setRightDialogVisible: false,
      // 编辑的对象
      editRoleForm: {},
      // 添加的对象
      addRoleForm: {
        roleName: '',
        roleDesc: ''
      },
      // 编辑的校验对象
      editRoleRules: {
        roleName: [{ required: true, message: '角色不能为空', trigger: 'blur' }]
      },
      // 添加用户校验对象
      addRoleRules: {
        roleName: [{ required: true, message: '角色不能为空', trigger: 'blur' }]
      },
      // 查询对象
      roleSearch: '',
      // 树形控件的绑定对象,一个模型
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      //  默认被选中的id数组
      defkeys: [],
      // 被点击的角色id
      roleId: 0

    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error("请求数据失败")
      this.rolesList = res.data
    },

    // 打开编辑对话框，并通过id查找对应角色
    async showEditRole (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$$message.error('请求数据失败!')
      this.editRoleForm = res.data
      this.editDialog = true
      console.log(res)
    },
    // 提交修改请求
    editFormConfirm () {
      this.$refs.editRoleFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`roles/${this.editRoleForm.roleId}`, this.editRoleForm)
        if (res.meta.status !== 200) return this.$message.error('修改角色失败')
        this.$message.success("修改成功")
        this.editDialog = false
        this.getRolesList()
      })
    },
    // 打开添加用户对话框
    showAddDialog () {
      this.addDialog = true
    },
    // 关闭后出发的更新清楚表单事件
    addDialogClosed () {
      this.$refs.addRoleFormRef.resetFields()
    },
    // 添加角色确定
    addFormConfirm () {
      this.$refs.addRoleFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('roles', this.addRoleForm)
        if (res.meta.status !== 201) return this.$message.error('添加失败')
        this.$message.success('添加成功')
        this.addDialog = false
        this.getRolesList()
      })
    },
    // 删除角色
    async deleteRole (id) {
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      this.getRolesList()
    },
    // 通过id删除对应的权限
    async removeRightById (role, rightId) {
      //  弹框是否删除
      const confirmResult = await this.$confirm('确定删除此权限吗', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((error) => error)
      if (!confirmResult) return this.$message.error("删除失败")
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败')
      this.$message.success('删除权限成功')
      // 把返回的结果挂载到最外层循环的children中 防止整个列表的刷新
      role.children = res.data
    },
    // 展示分配权限对话框
    async showSetRightDialog (node) {
      // 获取所有权限的数据
      this.roleId = node.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.message.error('获取权限数据失败')
      this.rightsList = res.data
      this.getLeafKeys(node, this.defkeys)
      console.log(this.defkeys)
      this.setRightDialogVisible = true
    },
    // 递归函数，获取节点全部的3级权限
    getLeafKeys (node, arr) {
      //  如果当前没有包含子节点，为3级节点，直接保存id
      if (!node.children) {
        arr.push(node.id)
      } else {
        node.children.forEach(item => this.getLeafKeys(item, arr))
      }
    },
    setRightDialogClose () {
      this.defkeys.length = 0
    },
    // 发送所有权限ID
    async allRights () {
      var rights = [
        ...this.$refs.rightsRef.getCheckedKeys(),
        ...this.$refs.rightsRef.getHalfCheckedKeys()
      ]
      rights = rights.toString()
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: rights })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.$message.success('请求成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>
<style lang="less" scoped>
.el-tag {
  margin: 10px 8px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eeeeee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
