<template>
  <el-container class="index">
    <el-header style="height: 50px;">
      <div class="set-left">
        <div style="float: left;">
          <img src="/static/img/logo.png" style="width: 30px;height: 30px;"/>
          <span style="margin-left: 10px;color: #fff;font-size: 14px;">vue-element-super</span>
        </div>
        <div @click="monitorMenu" class="monitor-menu">
          <i class="iconfont icon-list" style="font-size: 20px;"></i>
        </div>
      </div>
      <div class="index-header-list set-right">
        <ul>
          <li>
            <!-- 用户消息提示 -->
            <el-popover
              :visible-arrow="false"
              placement="top-start"
              offset="200"
              trigger="hover">
              <div class="hint">
                <div v-if="getUserMessage.length > 0">
                  <div class="hint-content" v-for="item in getUserMessage">
                    <div style="color: #00C1DE;font-size: 10px;">{{ item.hint }}</div>
                    <div style="margin-bottom: 10px;font-size: 10px;margin-top: 2px;">{{ item.time }}</div>
                  </div>
                </div>
                <div class="hint-content" v-else>
                  <div style="color: #00C1DE;font-size: 10px;">暂无消息</div>
                </div>
              </div>
              <div slot="reference" style="height: 50px;">
                <i class="iconfont icon-xiaoxi" style="font-size: 26px;margin-top: 20px;"></i>
                <sup class="index-sup">{{ getUserMessage.length }}</sup>
              </div>
            </el-popover>
          </li>
          <li>
            <!-- 用户个人信息 -->
            <el-popover
              :visible-arrow="false"
              placement="top-start"
              popper-class="index-popper-class"
              trigger="hover">
              <div class="hint">
                <ul class="index-hint-content">
                  <li @click="gotoHome">
                    首页
                  </li>
                  <li @click="gotoGithub">
                    项目地址
                  </li>
                  <li @click="exitLogin">
                    退出登录
                  </li>
                </ul>
              </div>
              <div slot="reference">
                <img src="/static/img/user.jpg" style="width: 28px;border-radius: 50%;height: 28px;margin-right: 5px;"/>{{ this.loginUser.user_name }}
              </div>
            </el-popover>
          </li>
        </ul>
      </div>
    </el-header>
    <el-container class="el-container-center">
      <el-scrollbar wrap-class="scrollbar-wrapper" style="overflow-x: hidden;background-color: #1f2d3d;height: 100%;">
        <el-aside style="width: auto;overflow-x: hidden;">
          <el-menu router
                   class="el-menu-vertical-demo"
                   text-color="rgb(191, 203, 217)"
                   active-text-color="rgb(64, 158, 255)"
                   :default-active="this.$store.getters.currentMenu.active"
                   :collapse="isCollapse">
            <template v-for="item in userMenuData">
              <el-submenu v-if="item.children.length !== 0" :index="item.url" :key="item.id">
                <template slot="title">
                  <i :class="item.icon"></i>
                  <span style="margin-left: 5px;">{{item.name}}</span>
                </template>
                <el-menu-item v-for="itemChild in item.children" :index="itemChild.url" :key="itemChild.id" style="padding-left: 25px;">
                  　　{{itemChild.name}}
                </el-menu-item>
              </el-submenu>
              <el-menu-item v-else :index="item.url" :key="item.id">
                <i :class="item.icon"></i>
                <span style="margin-left: 5px;">{{item.name}}</span>
              </el-menu-item>
            </template>
          </el-menu>
        </el-aside>
      </el-scrollbar>
      <el-main v-bind:style="{ width: elMainWidth+'px' }">
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
  import {getMenu} from '@/apis/menu'
  import {logout} from "../../apis/user";
  import {getTree} from "../../utils/menu"
  import Cookies from 'js-cookie'

  export default {
    name: "index",
    data() {
      return {
        elMainWidth:document.body.scrollWidth-200,
        userMenuData: null,
        isCollapse: false,
        loginUser:{},
        getUserMessage: [
          {
            hint: "有变更，可在跟踪查看",
            time: "2018-06-18 08:16",
          },
          {
            hint: "有变更，可在跟踪查看",
            time: "2018-06-18 08:16",
          }
        ],
      }
    },
    watch: {
      $route (to, from) {
        // 执行ajax请求，但只希望在进入时请求，离开时不希望进行请求。
        this.$store.commit('currentMenu', to.path);
      }
    },
    mounted: function () {
      this.loginUser = Cookies.getJSON("user");
      this.getUserMenu();
    },
    methods: {
      async getUserMenu () {
        let menudata = await getMenu();
        this.userMenuData = getTree(menudata, {id: "id", pid: "pid", rootPid: 0});
        if(this.loginUser.user_name === "admin"){
          this.userMenuData.unshift({"id": 0, "name": "首页", "pid": 0, "url": "/home/admin", "icon": "iconfont icon-home", "children": []});
        }else{
          this.userMenuData.unshift({"id": 0, "name": "首页", "pid": 0, "url": "/home/guest", "icon": "iconfont icon-home", "children": []});
        }
        this.$store.commit('userMenuPermissions', this.userMenuData);
        this.$store.commit('currentMenu', this.$router.currentRoute.path);
      },
      //退出登录
      exitLogin: function () {
        this.$confirm('确认退出登录?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.loginOut();
        }).catch(() => {

        });
      },
      async loginOut() {
        await logout();
        Cookies.remove("user");
        Cookies.remove("token");
        this.$router.push('/login');
      },
      //监听菜单缩放
      monitorMenu(){
        if(this.isCollapse){
          this.elMainWidth = 63;
        }else{
          this.elMainWidth = 200;
        }
        this.isCollapse = !this.isCollapse;
      },
      gotoHome:function () {
        if(this.loginUser.user_name === 'admin'){
          this.$router.push({path:'/home/admin'});
        }else if(this.loginUser.user_name === 'guest'){
          this.$router.push('/home/guest');
        }else{
          this.$router.push('/403');
        }
      },
      gotoGithub:function () {
        window.location.href = 'https://github.com/ljphilp/vue-element-super';
      }
    }
  }
</script>

<style>
  .index{
    height: 100%;
  }
  .index .el-header{
    background-color: #373d41;
    color: #333;
    text-align: center;
    height: 50px;
    line-height: 50px;
  }
  .index .el-header .index-sup{
    background-color: #f56c6c;
    position: relative;
    top: -10px;
    left: -8px;
    border-radius: 8px;
    color: #fff;
    display: inline-block;
    font-size: 12px;
    height: 15px;
    line-height: 15px;
    padding: 0 6px;
    text-align: center;
    white-space: nowrap;
    border: 1px solid #fff;
  }
  .index .el-header .monitor-menu{
    float: left;
    margin-left: 20px;
    width: 60px;
    color: #fff;
  }
  .index .el-header .monitor-menu:hover{
    background-color: #2a2f32;
    cursor: pointer;
  }

  /* 头部 */
  .index .index-header {
    height: 50px;
    line-height: 50px;
    width: 100%;
    background-color: #373d41;
    position: fixed;
    top: 0;
    left: 0;
    padding: 0 0 0 20px;
    box-sizing: border-box;
    font-size: 14px;
  }

  .index .el-header .index-header-list ul {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  .index .el-header .index-header-list ul li {
    margin: 0;
    padding: 0 15px;
    text-align: center;
    color: white;
    float: left;
    height: 50px;
    line-height: 50px;
    border-right: 1px solid #2a2f32;
    font-size: 14px;
  }
  .index .el-header .index-header-list ul li:first-child{
    border-left: 1px solid #2a2f32;
  }

  .index .el-header .index-header-list ul li:hover {
    background-color: #2a2f32;
  }

  .index .el-header .index-header-list ul li:last-child {
    border: none;
  }
  .index-popper-class{
    width: 130px;
    min-width: 130px;
    margin: 0;
    padding: 0;
  }
  .el-badge__content .el-badge__content--undefined .is-fixed{
    top:12px;
  }
  .el-badge__content{
    height: 15px;
    line-height: 15px;
    padding: 0 5px;
    font-size: 10px;
  }
  .index-hint-content{
    margin: 0;
    padding: 0;
  }
  .index-hint-content li{
    margin: 0;
    padding: 6px 20px;
  }
  .index-hint-content li:hover{
    color: #00C1DE;
    cursor: pointer;
  }

  .index .el-container-center{
    height: calc(100% - 50px);
  }
  .index .el-container-center .el-scrollbar__wrap{
    overflow-x: hidden;
  }
  .index .el-container-center .el-aside {
    color: #333;
    text-align: center;
    line-height: 200px;
  }
  .index .el-container-center .el-aside .el-menu{
    border: none;
  }

  /* 菜单样式 */
  .index .el-menu-vertical-demo:not(.el-menu--collapse) {
    height: 100%;
    min-height: 100%;
    width: 200px;
    float: left;
    text-align: left;
    overflow-x: hidden;
    box-sizing: border-box;
    background-color: #1f2d3d !important;
  }
  .el-menu li{
    background: rgb(38, 52, 69) !important;
  }
  .el-menu li:hover{
    background-color: #1f2d3d !important;
  }
  .index .el-menu--collapse{
    height: 100%;
    background-color: #1f2d3d !important;
    float: left;
  }
  .index .el-submenu__title {
    background-color: rgb(48, 65, 86) !important;
  }

  .index .el-submenu__title:hover {
    background: rgb(38, 52, 69) !important;
  }

  .index .el-menu-item {
    background-color: rgb(48, 65, 86) !important;
    padding-left: 20px;
  }

  .index .el-menu-item:hover {
    background: rgb(38, 52, 69) !important;
  }

  .index .el-submenu .el-menu-item {
    background-color: #1f2d3d !important;
    padding-left: 20px;
  }

  .index .el-submenu .el-menu-item:hover {
    background-color: #001528 !important;
  }

  .index .el-container-center .el-main {
    padding: 0;
    margin: 0;
    background-color: #fff;
    overflow-x: hidden;
  }

</style>
