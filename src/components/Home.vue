<template>
    <el-container class="home_container">
    <el-header>
        <div class="left">
            <img src="../assets/heima.png" alt="">
            <h3>电商管理后台</h3>
        </div>
        <el-button type="info" @click="logout">退出</el-button>
        </el-header>
    <el-container>
        <el-aside :width="isCollapse?'64px':'200px'" >
            <div class="toggle-button" @click="toggleCollapse">|||</div>
            <el-menu
            background-color="#333744"
            text-color="#fff"
            active-text-color="blue"
            unique-opened
            :collapse="isCollapse"
            :collapse-transition="false"
            router
            :default-active="activePath"
            >
                <el-submenu :index="item.id+''" v-for="item in list" :key="item.id">
                    <template slot="title">
                        <i :class="iconsObj[item.id]"></i>
                        <span>{{item.authName}}</span>
                    </template>
                    <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id" 
                    @click="saveNavState(subItem.path)">
                        <template slot="title">
                            <i class="el-icon-location"></i>
                            <span>{{subItem.authName}}</span>
                        </template>
                    </el-menu-item>
                </el-submenu>
            </el-menu>
        </el-aside>
        <el-main>
            <router-view></router-view>
        </el-main>
    </el-container>
</el-container>
</template>
<script>
export default {
    name: 'home',
    created(){
        this.getMenuList()
        this.activePath = window.sessionStorage.getItem('activePath')
    },
  data() {
    return {
        list: [],
        iconsObj: {
            '125': 'iconfont icon-user',
            '103':'iconfont icon-tijikongjian',
            '101':'iconfont icon-shangpin',
            '102':'iconfont icon-danju',
            '145':'iconfont icon-baobiao'
        },
        isCollapse: false,
        activePath: '',
    };
  },
  methods: {
    logout() {
      window.sessionStorage.clear();
      this.$router.push("/login");
    },
    async getMenuList(){
        const {data:res} = await this.$http.get('menus')
        if (res.meta.status!=200) {
            return this.$message.error('获取列表失败')
        }
        this.list = res.data
    },
    toggleCollapse(){
        this.isCollapse = !this.isCollapse
    },
    saveNavState(activePath) {
        sessionStorage.setItem('activePath','/' + activePath)
        this.activePath = '/' + activePath
    }
  }
};
</script>
<style scoped lang="less">
.home_container {
  height: 100%;
  .el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-left: 0;
    .left {
      display: flex;
      align-items: center;
      color: #fff;
      font-size: 20px;
      img {
        margin-right: 10px;
      }
      h3 {
        font-weight: normal;
        font-family: "Courier New", Courier, monospace;
      }
    }
  }
  .el-aside {
    background-color: #333744;
  }
  .el-main {
    background-color: #eaedf1;
  }
  .el-aside .el-menu {
      border-right: none;
  }
  .toggle-button{
    background-color: #4A5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    cursor: pointer;
    letter-spacing:.2em;
  }

}
</style>