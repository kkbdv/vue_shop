<template>
  <el-container class="el-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/homeL.jpg" alt="" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout"> 退出 </el-button>
    </el-header>
    <el-container>
      <!-- 页面侧边栏 -->
      <el-aside :width="collapseFlag ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">隐藏</div>
        <!-- 侧边栏菜单区 -->
        <el-menu
          :default-active="defaultPath"
          :collapse="collapseFlag"
          background-color="#dcdfe6"
          active-text-color="orange"
          :unique-opened="true"
          :collapse-transition="false"
          :router="true"
        >
          <!-- 一级菜单 -->
          <el-submenu
            :index="item.id + ' '"
            v-for="item in menulist"
            :key="item.id"
          >
            <!-- 一级菜单的模板区域 -->
            <template slot="title">
              <i :class="iconMenu[item.id]"></i>
              <span>{{ item.authName }}</span>
            </template>
            <!-- 二 级菜单的模板区域 -->
            <el-menu-item
              @click="saveNavState('/' + subItem.path)"
              :index="'/' + subItem.path"
              v-for="subItem in item.children"
              :key="subItem.id"
            >
              <i class="el-icon-setting"></i>
              <span slot="title">导航四</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 页面主题区域 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {

  data () {
    return {
      //  左侧菜单数据
      menulist: [],
      // 左侧菜单图标
      iconMenu: {
        125: 'el-icon-delete-solid',
        103: 'el-icon-platform-eleme',
        101: 'el-icon-s-goods',
        102: 'el-icon-zoom-out',
        145: 'el-icon-s-help'
      },
      collapseFlag: false,
      defaultPath: ''
    }
  },
  created () {
    this.getMenuList()
    this.defaultPath = window.sessionStorage.getItem('defaultPath')
    this.$router.push(this.defaultPath)
    //  如果有激活地址，跳转到激活地址，没有则跳到欢迎页面
    // if (this.defaultPath) { || '/home'
    //   this.$router.push(this.defaultPath)
    // } else {
    //   this.$router.push('/home')
    // }
  },
  methods: {

    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    //  获取所有菜单
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
    },
    // 水平折叠菜单
    toggleCollapse () {
      this.collapseFlag = !this.collapseFlag
    },
    // 保存连接的激活状态
    saveNavState (e) {
      this.defaultPath = e
      window.sessionStorage.setItem('defaultPath', e)
    }
  }
}
</script>

<style lang="less" scoped>
.el-header {
  background: #409eff;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #fff;
  font-size: 20px;
  img {
    width: 80px;
    vertical-align: middle;
    border-radius: 50%;
  }
  span {
    margin-left: 15px;
  }
}
.el-aside {
  background-color: #dcdfe6;
}
.el-main {
  background-color: #fff;
}
.el-container {
  height: 100%;
}
.el-menu {
  border: 0;
}
.toggle-button {
  background-color: white;
  text-align: center;
  font-size: 14px;
  padding: 5px 0;
  cursor: pointer;
}
</style>
