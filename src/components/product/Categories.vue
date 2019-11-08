<template>
    <div>
        <!-- 面包屑 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card>
            <el-button type="primary" @click="addCate">添加分类</el-button>
            <tree-table :data="catelist"
            :columns="columns"
            :expand-type="false"
            :selection-type="false"
            border stripe show-index index-text="#"
            >
            <template slot="isDeleted" slot-scope="scope">
                <i class="el-icon-success" style="color:green;" v-if="scope.row.cat_deleted==false"></i>
                <i class="el-icon-error" style="color:red;" v-else></i>
            </template>
            <template slot="level" slot-scope="scope">
                <el-tag size="mini" v-if="scope.row.cat_level==0">一级</el-tag>
                <el-tag type="success" size="mini" v-else-if="scope.row.cat_level==1">二级</el-tag>
                <el-tag type="info" size="mini" v-else>三级</el-tag>
            </template>
            <template slot="option" slot-scope="scope">
                
                <el-button type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
                <el-button type="danger" size="mini" icon="el-icon-delete">删除</el-button>
            </template>
            </tree-table>
            <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[1,2, 3, 4]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
            </el-pagination>
        </el-card>
        <el-dialog
        title="提示"
        :visible.sync="addCateDialogVisible"
        width="30%"
        @close="resetCateForm"
        >
        <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
            <el-form-item label="分类名称:" prop="cat_name">
                <el-input v-model="addCateForm.cat_name"></el-input>
            </el-form-item>
            <el-form-item label="父级分类:" prop="cat_name">
                <el-cascader
                v-model="selectedKeys"
                :options="parentCateList"
                :props="{ expandTrigger: 'hover',value:'cat_id',label:'cat_name',checkStrictly:true }"
                clearable
                @change="cascaderChange"
                >
                </el-cascader>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addCateDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="saveCate">确 定</el-button>
        </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
    created(){
      //获取商品分类数据
      this.getCateList()
    },
    data () {
        return {
            // 查询条件
            queryInfo:{
                type:3,
                pagenum:1,
                pagesize:2
            },
            //商品分类的数据列表，默认为空
            catelist:[],
            addCateDialogVisible:false,
            total: 0,
            columns:[
                {
                    prop: 'cat_name',
                    label: '分类名称'
                },
                {
                    prop: 'cat_deleted',
                    label: "是否有效",
                    //表示将当前列定义为模板列
                    type: "template",
                    //表示当前这一行使用模板的名称
                    template: "isDeleted"
                },
                {
                    prop: 'cat_level',
                    label: '排序',
                    type: 'template',
                    template: 'level'
                },
                {
                    label: '操作',
                    type: 'template',
                    template: 'option'
                }
            ],
            addCateForm: {
                cat_name: '',
                cat_pid: 0,
                cat_level:0
            },
            addCateFormRules: {
                cat_name: [
                    {required:true,message:'分类名称不能为空',trigger:'blur'},
                    {min:3,max:15,message:'长度在3到15之间',trigger:'blur'}
                ]
            },
            parentCateList:[],
            selectedKeys:[]
        }
    },
    methods:{
        async getCateList(){
            const {data:res} = await this.$http.get('categories',{params:this.queryInfo})
            if(res.meta.status != 200)return this.$message.error('获取商品分类失败')
            //数据列表
            this.catelist = res.data.result 
            //数据总条数
            this.total = res.data.total 
        },
        handleSizeChange(pagesize){
            this.queryInfo.pagesize = pagesize
            this.getCateList()
        },
        handleCurrentChange(pagenum){
            this.queryInfo.pagenum = pagenum
            this.getCateList()
        },
        addCate(){
            this.addCateDialogVisible = true
            this.getParentsCateList()
        },
        async getParentsCateList(){
            const {data:res} = await this.$http.get('categories',{params:{type:2}})
            if (res.meta.status!=200) {
                return this.$message.error('获取失败')
            }
            this.parentCateList = res.data
        },
        cascaderChange(){
            if (this.selectedKeys.length == 0) {
                this.addCateForm.cat_pid = 0
            }else {
                this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length-1]
            }
            this.addCateForm.cat_level = this.selectedKeys.length
        },
        resetCateForm(){
            this.$refs.addCateFormRef.resetFields()
            this.selectedKeys = []
            this.addCateForm.cat_pid = 0
            this.addCateForm.cat_level = 0
        },
        async saveCate(){
            const {data:res} = await this.$http.post('categories',this.addCateForm)
            if (res.meta.status != 201) {
                return this.$message.error('添加失败')
            }
            this.getCateList()
            this.addCateDialogVisible = false
        }
    }
}
</script>
<style scoped lang="less">
    
</style>
<style lang="less">
    .el-cascader-menu {
        height: 240px;
    }
</style>