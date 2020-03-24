<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog()">添加分类</el-button>
        </el-col>
      </el-row>
      <tree-table
      class="treeTable"
      :data="cateList"
      :columns="columns"
      :selection-type="false"
      :expand-type="false"
      show-index
      index-text="#"
      border
      :show-row-hover="false"
      >
      <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" style="color:lightgreen" v-if="scope.row.cat_deleted === false"></i>
          <i class="el-icon-error" style="red" v-else></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="mini" v-else-if="scope.row.cat_level === 1" type="success">二级</el-tag>
          <el-tag size="mini" v-else type="warning">三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" >
          <el-button type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
          <el-button type="danger" size="mini" icon="el-icon-delete">删除</el-button>
        </template>
      </tree-table>
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[3,5,10,15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>
    <!-- 添加按钮对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
      >
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <!-- options指定属性源 -->
          <!-- props指定级联选择器的配置对象 -->
           <el-cascader
              clearable
              v-model="selectedKeys"
              :options="parentCateList"
              :props="cascaderProps"
              @change="parentCateChanged"></el-cascader>
        </el-form-item>
        </el-form>    
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data(){
    return{
      // 查询条件
      queryInfo:{
        type:3,
        pagenum:1,
        pagesize:5
      },
      // 商品分类列表
      cateList:[],
      total:0,
      // 为column指定列
      columns:[
        {
          label:'分类名称',
          prop:'cat_name'
        },
        {
          label:'是否有效',
          type:'template',
          template:'isok'
        },
        {
          label:'排序',
          type:'template',
          template:'order'
        },
        {
          label:'操作',
          type:'template',
          template:'opt'
        }
      ],
      addCateDialogVisible:false,
      addCateForm:{
        cat_name:'',
        // 父级分类的id
        cat_pid:0,
        // 分类的等级
        cat_level:0
      },
      // 添加表单校验规则
      addCateFormRules:{
        cat_name:[
          {required: true, message: '请输入分类名称', trigger: 'blur'}
        ]
      },
      // 父级分类列表
      parentCateList:[],
      // 为级联选择器的props指定配置对象
      cascaderProps:{
        checkStrictly:true,
        expandTrigger: 'hover',
        value:'cat_id',
        label:'cat_name',
        children:'children'
      },
      // 选中的父级分类的id数组
      selectedKeys:[]
    }
  },
  created() {
    this.getCateList()
  },
    mounted () {
        setInterval(function () {
          document.querySelectorAll('.el-cascader-node__label').forEach(el => {
            el.onclick = function () {
              if (this.previousElementSibling) this.previousElementSibling.click()
            }
          })
        }, 1000)
      },

  methods: {
    // 获取商品分类列表
    async getCateList(){
      const {data:res} = await this.$http.get('categories',{params:this.queryInfo})
      if(res.meta.status !== 200) return this.$message.error('获取商品分类失败')
      console.log(res.data.result)
      this.cateList = res.data.result
      // 总数
      this.total = res.data.total
    },
    // 监听pageSize改变
    handleSizeChange(newSize){
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pageNum改变
    handleCurrentChange(newPage){
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮弹出添加对话框
    showAddCateDialog(){
      // 获取父级分类列表
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类数据
    async getParentCateList(){
      const {data:res} = await this.$http.get('categories',{params:{type:2}})
      if(res.meta.status !== 200) return this.$message.error('获取父级分类数据失败')
      this.parentCateList = res.data
      // console.log("###########")
      // console.log(this.parentCateList);
    },
    // 选择项发生变化触发这个函数
    parentCateChanged(){
      // console.log(this.selectedKeys)
      // 如果selectedKeys.length>0说明有父级分类
      if(this.selectedKeys.length>0){
        this.addCateForm.cat_pid=this.selectedKeys[this.selectedKeys.length-1]
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      }
      else{
         this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 点击按钮，添加新的分类
    addCate(){
       this.$refs.addCateFormRef.validate(async valid => {
         if(!valid) return
         const {data:res} = await this.$http.post('categories',this.addCateForm)
         if(res.meta.status !== 201){
           return this.$message.error('添加分类失败')
         }
         this.$message.success('添加分类成功')
         this.getCateList()
         this.addCateDialogVisible = false
       })
    },
    addCateDialogClosed(){
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    }

  },
}
</script>
<style lang="less" scoped>
  .treeTable{
    margin-top: 15px;
  }
  .el-cascader{
    width: 100%;
  }

  

</style>