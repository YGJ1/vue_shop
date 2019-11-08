<template>
    <div>
         <!-- 面包屑 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary">添加角色</el-button>
                </el-col>
            </el-row>
            <el-table :data="rolelist" border stripe>
                <!-- 展开行 -->
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row :class="['bdbottom',i1===0?'bdtop':'']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                            <!-- 渲染一级权限 -->
                            <el-col :span="5">
                                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 渲染二级和三级权限 -->
                            <el-col :span="19">
                                <el-row :class="[i2===0?'':'bdtop']" v-for="(item2,i2) in item1.children" :key="item2.id">
                                    <el-col :span="6">
                                        <el-tag closable @close="removeRightById(scope.row,item2.id)" type="success">{{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag closable @close="removeRightById(scope.row,item3.id)" type="warning" v-for="(item3) in item2.children" :key="item3.id">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <!-- 索引行 -->
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" width="300px">
                    <template slot-scope="scope">
                        <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                        <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                        <el-button @click="showSetRightDialog(scope.row)" size="mini" type="warning" icon="el-icon-setting">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <el-dialog
            title="分配权限"
            :visible.sync="setRightDialogVisible"
            width="50%"
            @close="setRightDialogClosed"
            >
            <el-tree :data="rightlist" :props="treeProps" show-checkbox
            default-expand-all
            node-key="id"
            :default-checked-keys="defkeys"
            ref="mytree"
            ></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="allocateRights">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
    name: 'roles',
    created(){
        this.getRolesList()
    },
    data () {
        return {
            rolelist:[],
            setRightDialogVisible: false,
            rightlist: [],
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            defkeys: [],
            roleId: ''
        };
    },
    methods:{
        async getRolesList(){
            const {data:res} = await this.$http.get('roles')
            if (res.meta.status != 200) {
                return this.$message.error('获取角色列表失败')
            }
            this.rolelist = res.data
        },
        async removeRightById(role,rightId){
            const confirmReslut = await this.$confirm('此操作将永久删除该文件，是否继续?','提示',{
                confirmButtonText:'确定',
                cancelButtonText:'取消',
                type:'warning'
            }).catch(err=>err)
            if (confirmReslut != 'confirm') {
                return this.$message.info('取消了删除')
            }
            const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
            if (res.meta.status != 200) {
                return this.$message.error('删除失败')
            }
            role.children = res.data
        },
        async showSetRightDialog(role){
            this.roleId = role.id
            const {data:res} = await this.$http.get('rights/tree')
            if(res.meta.status!=200)return this.$message.error('获取权限数据失败!')
            this.rightlist = res.data 
            this.getLeafKeys(role,this.defkeys)
            this.setRightDialogVisible = true
        },
        getLeafKeys(node,arr) {
            if (!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach(item => this.getLeafKeys(item,arr));
        },
        setRightDialogClosed(){
            this.defkeys = []
        },
        async allocateRights(){
            const keys = [
                ...this.$refs.mytree.getCheckedKeys(),
                ...this.$refs.mytree.getHalfCheckedKeys()
            ]
            const idStr = keys.join(',')
            const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
            if (res.meta.status != 200) {
                return this.$message.error('分配权限失败')
            }
            this.$message.success('分配权限成功')
            this.getRolesList()
            this.setRightDialogVisible = false
        }
    }
}
</script>
<style scoped lang="less">
    .el-tag {
        margin: 7px
    }
    .bdbottom {   
        border-bottom: 1px solid #ccc;
    }
    .bdtop {
        border-top: 1px solid #ccc;
    }
</style>