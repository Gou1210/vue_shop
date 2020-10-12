<template>

    <el-container class="home-container">
        <!-- 头部 -->
  <el-header>
      <div >
          <img src="../assets/window.png" alt="">
          <span>后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
      </el-header>
  <el-container>
      <!-- 侧边栏 -->
    <el-aside :width="isCollapse?'64px':'200px'">
        <div @click="toggleButton" class="toggle-button">|||</div>
        <el-menu unique-opened :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath"
      class="el-menu-vertical-demo"
      background-color="#323744"
      text-color="#fff"
      active-text-color="#3a88fd">
      <!-- 一级菜单 -->
      <el-submenu v-for="(item,index) in menuList" :v-key="item.id" :key="item.id" :index="item.id+''">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span> {{item.authName}}</span>
        </template>
<!-- 二级菜单 -->
          <el-menu-item @click="saveNavState('/'+subItem.path)" v-for="(subItem,subIndex) in item.children" :key="subItem.id" :index="'/'+subItem.path" >        
              <template slot="title">
          <i class="el-icon-menu"></i>
          <span>{{subItem.authName}}</span>
        </template>
        </el-menu-item>
      </el-submenu>
     
    </el-menu>
    </el-aside>
<!-- 主体 -->
    <el-main>
        <router-view></router-view>
    </el-main>
  </el-container>
</el-container>
</template>

<script>
export default {
    data(){
        return{
            menuList:[],
            isCollapse:false,
            activePath:''
        }
    },
    created(){
        this.getMenuList()
        this.activePath = window.sessionStorage.getItem('activePath')
    },
    methods:{
        async getMenuList() {
            const {data:res} = await this.$http.get('menus')
            this.menuList = res.data
        },
        logout(){
            window.sessionStorage.clear()
            this.$router.push('/login')
        },
        toggleButton(){
            this.isCollapse = !this.isCollapse
        },
        saveNavState(e){
            window.sessionStorage.setItem('activePath',e)
            this.activePath = e
        }
    }
}
</script>

<style lang="less" scoped>
.home-container{
    width: 100%;
    height: 100%;
}
.el-header{
    background-color: #373c41;
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #eaedf2;
    div{
        display: flex;
        align-items: center;
    }
    img{
        width: 20%;
    }
}
.el-aside{
    background-color: #323744;
}
.el-main{
    background-color: #eaedf2;
}
.toggle-button{
    width: 100%;
    height: 30px;
    background-color: #666;
    text-align: center;
    color: #fff;
}
</style>