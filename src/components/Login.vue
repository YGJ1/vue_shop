<template>
    <div class="login_container">
        <div class="login_box">
            <!-- 头像区域 -->
            <div class="avatar_box">
                <img src="../assets/logo.png" alt="">
            </div>
            <!-- 表单区域 -->
            <el-form :model="loginForm" label-width="0px" class="login_form">
                <el-form-item>
                    <el-input prefix-icon="iconfont icon-user" v-model="loginForm.username"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-input prefix-icon="iconfont icon-3702mima" v-model="loginForm.password" type="password"></el-input>
                </el-form-item>
                <el-form-item class="btn_box">
                    <el-button type="primary">登录</el-button>
                    <el-button type="info">重置</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>
<script>
export default {
    name: 'login',
    data () {
        return {
            loginForm: {
                username: '',
                password: ''
            }
        };
    },
    methods:{
    
    }
}
</script>
<style scoped lang="less">
    .login_container {
        background-color: #2b4b6b;
        height: 100%;
        
        .login_box {
            width: 450px;
            height: 300px;
            background-color: #fff;
            border-radius: 3px;
            position: absolute;
            left: 50%;
            top: 50%;
            transform: translate(-50%,-50%);
            .avatar_box {
                width: 130px;
                height: 130px;
                border: 1px solid #eee;
                border-radius: 50%;
                padding: 10px;
                box-shadow: 0 0 10px #ddd;
                position: absolute;
                left: 50%;
                top: 0;
                transform: translate(-50%,-50%);
                background-color: #fff;
                img {
                    width: 100%;
                    height: 100%;
                    border-radius: 50%;
                    background-color: #eee;
                }
            }
        }
    }
    .login_form {
        position: absolute;
        left: 0;
        bottom: 0;
        width: 100%;
        box-sizing: border-box;
        padding: 0 20px;
        .btn_box {
            display: flex;
            justify-content: flex-end;
        }
    }
    
    
</style>