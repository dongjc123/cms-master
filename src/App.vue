<template>
  <div class="app">
    <!-- 头 -->
    <div class="header note" :style="note">
      <div class="title">
        <i class="fa fa-newspaper-o"></i> &nbsp;&nbsp;热点新闻资讯
      </div>
      <div class="info">
        <img class="photo" :src="user.userface" alt="">
        <div class="dropdown">
          <el-dropdown @command="handleCommand">
          <span class="el-dropdown-link">
            {{user.nickname}}<i class="el-icon-arrow-down el-icon--right"></i>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command='logout'>退出</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
        </div>
      </div>
    </div>
    <!-- 体 -->
    <div class="center">
      <!-- 左侧导航 -->
      <div class="left-nav">
        <ul>
          <li :class="{current:currentRoute=='/'}">
            <i class="fa fa-newspaper-o"></i>
            <router-link to='/'>首页</router-link>
            <i class="fa fa-angle-right"></i>
          </li>
          <li :class="{current:currentRoute=='/category'}">
            <i class="fa fa-bars"></i>
            <router-link to='/category'>栏目管理</router-link>
            <i class="fa fa-angle-right"></i>
          </li>
          <li :class="{current:currentRoute=='/article'}">
            <i class="fa fa-briefcase"></i>
            <router-link to='/article'>文章管理</router-link>
            <i class="fa fa-angle-right"></i>
          </li>
          <li :class="{current:currentRoute=='/user'}">
            <i class="fa fa-user"></i>
            <router-link to='/user'>用户管理</router-link>
            <i class="fa fa-angle-right"></i>
          </li>
          <li :class="{current:currentRoute.indexOf('/setting')>=0}">
            <i class="fa fa-cog"></i>
            <router-link to='/setting'>系统设置</router-link>
            <i class="fa fa-angle-right"></i>
          </li>
        </ul>

      </div>
      <!-- 右侧内容区 -->
      <div class="content">
        <div class="wrapper">
          <router-view></router-view>
        </div>
        
      </div>      
    </div>
    <!-- 底部 -->
   <!--  <div class="footer">
      <span><img src="./assets/国标.gif"></span>
      
      <span>huiasdhf</span>
    </div> -->
    
  </div>
</template>
<script>
  import axios from '@/http/axios'
  export default {
    data(){
      return {
        currentRoute:'/',
        user:{},
        note: { 
          backgroundImage: "url(" + require("./assets/login-bg.jpg") + ") ",
          backgroundPosition: "center center", 
          backgroundRepeat: "no-repeat", 
          backgroundSize: "cover", 
        },
      }
    },
    watch:{
      '$route':function(to,from){
        this.currentRoute = to.path;
      }
    },
    created(){
      this.currentRoute = this.$route.path;
      //从localStorage中获取用户信息，显示在右上角
      let user = JSON.parse(localStorage.getItem('user'));
      if(user && user.id){
        this.user = user;
      } else {
        window.vm.currentComponent = 'Login';
      }
    },
    methods:{
      handleCommand(command){
        if(command == 'logout'){
           axios.get('http://47.107.41.99:8888/logout')
           .then(()=>{
              //跳转到登录页
              window.vm.currentComponent = 'Login';
              //清理localStorage中的user
              localStorage.removeItem('user');
           });
        }
      }
    }
  }
</script>
<style>
  html {
    font: normal normal 12px '微软雅黑','Microsoft YaHei';
    color: #666
  }
  body , ul ,ol,dl ,p, h1,h2,h3 {
    margin: 0;
    padding: 0
  }
  ul , ol {
    list-style: none;
  }
  a {
    color: #666;
    text-decoration: none;
  }
  div {
    box-sizing: border-box;
  }
  .header {
    position: absolute;
    width: 100%;
    height: 80px;
    top: 0;
    background-image: url('./assets/login-bg.jpg');   
    padding: 0 4em; 
  }
  .header .title {
    color: #ffffff;
    line-height: 80px;
    font-size: 26px;
    float: left;
  }
  .header .info {
    text-align: right;
    cursor: pointer;
  }
  .header .info .photo {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    margin-top: 10px;
    margin-right: 10px;
  }
  .header .info .dropdown {
    float: right;
    line-height: 20px;
    margin-top: 30px;
  }
  .header .info .el-dropdown {
    font-size: 20px;
    color: #f0f0f0;
  }
  .center {
    position: absolute;
    top: 80px;
    bottom: 0;
    width: 100%;

  }
  .center > .left-nav {
    width: 180px;
    height: 100%;
    float: left;
    background-color: #025395;
  }
  .center > .left-nav > ul {

  }
  .center > .left-nav > ul > li{
    font-size: 18px;
    line-height: 3em;
    text-align: center;
    border-bottom: 1px solid #f0f0f0;
    position: relative;
  }
  .center > .left-nav > ul > li i.fa {
    position: absolute;
    top: 50%;
    margin-top: -.5em;
    color: #FFF;
  }
  .center > .left-nav > ul > li > a{
    color: #fff;
  }
  .center > .left-nav > ul > li i.fa:last-child {
    right: 1em;
  }
  .center > .left-nav > ul > li i.fa:first-child {
    left: 1em;
  }
  .center > .left-nav > ul > li.current {
    background-color: #f0f0f0;
  }
  .center > .left-nav > ul > li.current > i.fa{
    color: blue;
  }
  .center > .left-nav > ul > li.current > a{
    color: blue;

  }
  .center > .content {
    margin-left: 180px;
    height: 100%;
    background-color: #f0f0f0;
    padding: 1em 1em 0 1em;
    overflow-y: auto;
  }
  .center > .content > .wrapper {
    width: 100%;
    height: 100%;
    background-color: #ffffff;
    border-radius: 5px;
    /*position: absolute;*/
    padding: .5em;
  }
  .footer{
    width: 100%;
    height: 40px;
    text-align: center;
    line-height: 40px;
    background-color: #ccc;
    position: absolute;
    bottom: 0;
  }
  
  .footer span img{
    /*margin: 5px;*/
    /*line-height: 40px;*/
    margin-top: 5px;
    width: 25px;
    height: 25px;
  };
</style>