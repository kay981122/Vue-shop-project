<template>
  <el-container class="home-containter">
    <!--头部区域-->
    <el-header>
      <div>
        <img src="../assets/nike.jpg" alt="">
        <span>运动品牌商城后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!--页面主体区域-->
    <el-container>
      <!--侧边栏-->
      <el-aside :width="isCollapse ? '64px': '200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <!--侧边栏菜单-->
        <el-menu
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409EFF"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="activePath"
          >
          <!--一级菜单-->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <!--一级菜单的模板区域-->
            <template slot="title">
              <!--图标-->
              <i :class="iconsObj[item.id]"></i>
              <!--文本-->
              <span>{{item.authName}}</span>
            </template>
            <!--二级菜单-->
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavStatus('/'+subItem.path)">
              <template slot="title">
                <!--图标-->
                <i class="el-icon-menu"></i>
                <!--文本-->
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>

        </el-menu>
      </el-aside>
      <!--右侧内容主体-->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      // 左侧菜单数据
      menuList: [
        {
          id: 125,
          authName: '用户管理',
          path: 'users',
          order: 1,
          children: [{
            id: 110,
            authName: '用户列表',
            order: 1,
            path: 'users',
            children: []
          }]
        },
        {
          id: 103,
          authName: '权限管理',
          path: 'rights',
          order: 2,
          children: [{
            id: 111,
            authName: '角色列表',
            order: null,
            path: 'roles',
            children: []

          }, {
            id: 112,
            authName: '权限列表',
            order: null,
            path: 'rights',
            children: []

          }]
        },
        {
          id: 101,
          authName: '商品管理',
          path: 'goods',
          order: 3,
          children: [{
            id: 104,
            authName: '商品列表',
            order: 1,
            path: 'goods',
            children: []

          }, {
            id: 115,
            authName: '分类参数',
            order: null,
            path: 'params',
            children: []

          }, {
            id: 121,
            authName: '商品分类',
            order: 3,
            path: 'categories',
            children: []
          }]
        },
        {
          id: 102,
          authName: '订单管理',
          path: 'orders',
          order: 4,
          children: [{
            id: 107,
            authName: '订单列表',
            order: 1,
            path: 'orders',
            children: []

          }]
        },
        {
          id: 145,
          authName: '数据统计',
          path: 'reports',
          order: 5,
          children: [{
            id: 146,
            authName: '数据报表',
            order: null,
            path: 'reports',
            children: []

          }]
        }],
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      // 是否折叠
      isCollapse: false,
      // 被激活的链接地址
      activePath: ''
    }
  },
  created() {
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  name: 'Home.vue',
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 点击按钮切换菜单折叠与展开
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    // 保存链接的激活状态
    saveNavStatus(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style lang="less" scoped>
.el-header{
  background-color: black;
  display: flex;
  justify-content: space-between;
  padding: 0;
  align-items: center;
  color:#fff;
  font-size: 20px;
  > div{
    display: flex;
    align-items: center;
    span{
      margin-left: 15px;
    }
    img{
      height: 50px;
      width: 50px;
    }
  }
}
.el-aside{
  background-color: #333744;
  .el-menu{
    border-right: none
  }
}
.el-main{
  background-color: #eaedf1;
}
.home-containter{
  height: 100%;
}
.iconfont{
  margin-right: 10px;
}
.toggle-button{
  background-color: #4A5064;
  color: #fff;
  font-size: 10px;
  line-height: 24px;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
