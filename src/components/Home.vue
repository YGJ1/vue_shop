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
        <el-aside width="200px" >
            <el-menu
            background-color="#333744"
            text-color="#fff"
            active-text-color="blue"
            >
                <el-submenu :index="item.id+''" v-for="item in list" :key="item.id">
                    <template slot="title">
                        <i class="el-icon-location"></i>
                        <span>{{item.authName}}</span>
                    </template>
                    <el-menu-item :index="subItem.id+''" v-for="subItem in item.children" :key="subItem.id">
                        <template slot="title">
                            <i class="el-icon-location"></i>
                            <span>{{subItem.authName}}</span>
                        </template>
                    </el-menu-item>
                </el-submenu>
            </el-menu>
        </el-aside>
        <el-main>Main</el-main>
    </el-container>
</el-container>
</template>
<script>
export default {
    name: 'home',
    created(){
        this.getMenuList()
    },
  data() {
    return {
        list: []
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
        console.log(this.list)
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
}
</style>