<template>
    <div>
         <!-- 面包屑 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>分类参数</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card>
            <el-alert
            title="注意：只允许为第三级分类设置相关参数！"
            type="warning"
            effect="dark"
            >
            </el-alert>
            <div class="cateList_box">
                请选择商品分类:
                <el-cascader
                v-model="selectedKeys"
                :options="cateList"
                :props="{ expandTrigger: 'hover',value:'cat_id',label:'cat_name',children:'children' }"
                clearable
                @change="handleChange"
                >
                </el-cascader>
            </div>
            <el-tabs v-model="activeName" @tab-click="handleClick">
                <el-tab-pane label="动态属性" name="many">
                    <el-button type="primary" :disabled="isDisabled" @click="showAddDialog">添加属性</el-button>
                    <el-table :data="manyTableData" border stripe>
                        <el-table-column type="expand"></el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog">修改</el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <el-tab-pane label="静态属性" name="only">
                    <el-button type="primary" :disabled="isDisabled" @click="showAddDialog">添加属性</el-button>
                    <el-table :data="onlyTableData" border stripe>
                        <el-table-column type="expand"></el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog">修改</el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>
        <el-dialog
        :title="'添加'+currentTitle"
        :visible.sync="addAttrDialogVisible"
        width="30%"
        @close="resetForm"
        >
            <el-form :model="attrForm" :rules="attrFormRules" ref="attrFormRef" label-width="100px">
                <el-form-item :label="currentTitle" prop="attr_name">
                    <el-input v-model="attrForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addAttrDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveAttrs">确 定</el-button>
            </span>
        </el-dialog>
        <el-dialog
        :title="'修改'+currentTitle"
        :visible.sync="editAttrDialogVisible"
        width="30%"
        @close="resetEditForm"
        >
            <el-form :model="editAttrForm" :rules="attrFormRules" ref="editAttrFormRef" label-width="100px">
                <el-form-item :label="currentTitle" prop="attr_name">
                    <el-input v-model="editAttrForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editAttrDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEditAttrs">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
    name: 'params',
    created(){
        this.getCateList()
    },
    data () {
        return {
            cateList:[],
            selectedKeys: [],
            activeName:'many',
            onlyTableData: [],
            manyTableData: [],
            addAttrDialogVisible: false,
            editAttrDialogVisible:false,
            editAttrForm:{
                attr_name: ''
            },
            attrForm:{
                attr_name: ''
            },
            attrFormRules: {
                attr_name:[
                    {required:true,message:'属性名不能为空',trigger:'blur'}
                ]
            }
        };
    },
    methods:{
        async getCateList(){
            const {data:res} = await this.$http.get('categories')
            if (res.meta.status != 200) {
                return this.$message.error('获取分类列表失败');
            }
            this.cateList = res.data
        },
        handleChange(){
            this.getParamsData()
        },
        handleClick(){
            this.getParamsData()
        },
        async getParamsData(){
            if (this.selectedKeys.length <= 2) {
                return this.selectedKeys = [] 
            }
            const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:this.activeName}})
            if (this.activeName == 'only') {
                this.onlyTableData = res.data
            }else {
                this.manyTableData = res.data
            }
        },
        showAddDialog(){
            this.addAttrDialogVisible = true
        },
        resetForm(){
            this.$refs.attrFormRef.resetFields()
        },
        saveAttrs(){
            this.$refs.attrFormRef.validate(async valid => {
                console.log()
                const {data:res} = await this.$http.post(`categories/${this.cateId}/attributes`,{
                    attr_name:this.attrForm.attr_name,
                    attr_sel: this.activeName
                })
                if (res.meta.status != 201) {
                    return this.$message.error('添加属性失败')
                }
                this.addAttrDialogVisible = false
                this.getParamsData()
            })
        },
        saveEditAttrs(){

        },
        resetEditForm(){
            this.$refs.editAttrFormRef.resetFields()
        },
        showEditDialog(){
            this.editAttrDialogVisible = true
        }
    },
    computed: {
        isDisabled: function(){
            return this.selectedKeys.length != 3
        },
        cateId: function(){
            if (this.selectedKeys.length == 3) {
                return this.selectedKeys[2]
            }else {
                return null
            }
        },
        currentTitle(){
            if (this.activeName == 'only') {
                return '静态属性'
            }else {
                return '动态属性'
            }
        }
    }
}
</script>
<style scoped lang="less">
    .cateList_box {
        margin-top: 15px
    }
</style>