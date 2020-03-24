<template>
  <div>
        <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
       <el-alert show-icon title="注意：只允许为第三级分类设置相关参数" type="warning" :closed="false">
      </el-alert>

      <!-- 选择商品分类 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类级联选择框 -->
        <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cateProps"
            @change="handleChange"></el-cascader>
        </el-col>
      </el-row>
      <!-- tab页签区 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 添加动态参数面板 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button @click="addDialogVisible = true" :disabled="isBtnDisabled" type="primary" size="mini">添加参数</el-button>
        <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <template v-slot="scope">
                <el-tag @close="handleClose(i,scope.row)" closable v-for="(item,i) in scope.row.attr_vals" :key="i">{{item}}</el-tag>
              <!-- 添加tag文本框-->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button @click="showEditDialog(scope.row.attr_id)" size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                <el-button @click="removeParams(scope.row.attr_id)" size="mini" type="danger" icon="el-icon-delete">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态参数面板 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button @click="addDialogVisible = true" :disabled="isBtnDisabled" type="primary" size="mini">添加属性</el-button>
          <!-- 静态参数表格 -->
          <el-table :data="onlyTableData" border stripe>
            <el-table-column type="expand">
              <template v-slot="scope">
                <el-tag @close="handleClose(i,scope.row)" closable v-for="(item,i) in scope.row.attr_vals" :key="i">{{item}}</el-tag>
              <!-- 添加tag文本框-->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="属性名称"></el-table-column>
            <el-table-column label="操作">
              <template v-slot="scope">
                <el-button @click="showEditDialog(scope.row.attr_id)" size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                <el-button @click="removeParams(scope.row.attr_id)"  size="mini" type="danger" icon="el-icon-delete">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数对话框 -->
    <el-dialog
    @close="addDialogClosed"
      :title="'添加'+titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      >
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑对话框 -->
    <el-dialog
    @close="editDialogClosed"
      :title="'修改'+titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      >
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data(){
    return{
      cateList:[],
      // 级联选择框配置对象
      cateProps:{
        value:'cat_id',
        label:'cat_name',
        children:'children',
        expandTrigger: 'hover'
      },
      // 级联选择框双向绑定到的数组
      selectedCateKeys:[],
      // 激活页签名字
      activeName:'many',
      btnDisabledFlag :true,
      manyTableData:[],
      onlyTableData:[],
      // 添加对话框
      addDialogVisible:false,
      // 添加参数表单验证
      addForm:{},
      // 添加表单验证
      addFormRules:{
        attr_name:[
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      editDialogVisible:false,
      editForm:{},
      editFormRules:{
        attr_name:[
          { required: true, message: '请输入活动名称', trigger: 'blur' }
        ]
      },


    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList(){
      const {data:res} = await this.$http.get('categories')
      if(res.meta.status !== 200) return this.$message.error('获取商品分类失败')
      this.cateList = res.data
      console.log(this.cateList)
    },
    // 级联选择框选项变化触发
    handleChange(){
      this.getParamsData()
    },
    // 页签点击事件
    handleTabClick(){
      this.getParamsData()
      // console.log(this.activeName)
    },
    // 获取参数列表所有数据
    async getParamsData(){
      if(this.selectedCateKeys.length !== 3){
        this.selectedCateKeys = []
        return
      }
      const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{
        params:{sel:this.activeName}
      })
      if(res.meta.status !== 200)  return this.$message.error('获取参数列表失败')
      
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals?item.attr_vals.split(' '):[]
      // 控制按钮与文本框切换显示
        item.inputVisible = false
      // 文本框输入内容
        item.inputValue = ''
     })
      console.log(res.data)
      if(this.activeName === 'many'){
        this.manyTableData = res.data       
      }
      else{
        this.onlyTableData = res.data
      }
    },
    addDialogClosed(){
      this.$refs.addFormRef.resetFields()
    },
    addParams(){
      this.$refs.addFormRef.validate(async valid =>{
        if(!valid) return
        const {data:res} = await this.$http.post(`categories/${this.cateId}/attributes`,{
          attr_name:this.addForm.attr_name,
          attr_sel:this.activeName
        })
        console.log(res)
        if(res.meta.status !== 201) return this.$message.error('添加参数失败')
        this.$message.success('添加参数成功')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    async showEditDialog(attr_id){
      const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`,
      {params:{attr_sel:this.activeName}})
      if(res.meta.status !== 200) return this.$message.error('获取参数信息失败')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editDialogClosed(){
      this.$refs.editFormRef.resetFields()
    },
    // 点击按钮，修改参数信息
    editParams(){
      this.$refs.editFormRef.validate(async valid =>{
        if(!valid) return
        const {data:res} = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
        {attr_name:this.editForm.attr_name,attr_sel:this.activeName})
        if(res.meta.status !== 200) return this.$message.error('修改参数失败')
        this.$message.success('修改参数成功')
        this.getParamsData()
        this.editDialogVisible = false
      })
    },
    // 删除参数
    async removeParams(attr_id){
      const message = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
      if(message !== 'confirm') return this.$message.info('取消删除参数')
      const {data:res} = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
      if(res.meta.status !== 200) return this.$message.error('删除参数失败')
      this.getParamsData()
      this.$message.success('删除参数成功')
    },
    // 文本框失去焦点或enter
     handleInputConfirm(row){
      if(row.inputValue.trim().length === 0){
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      // 如果没有return，做后期处理
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveAttrVals(row)
    },
    // 点击按钮，展示文本输入框
    showInput(row){
      row.inputVisible = true
      // 让文本框自动或的焦点
      // $nextTick当页面重新渲染后，执行这个代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    // 将对attr_vals的操作存入数据库
    async saveAttrVals(row){

      // 发起请求，保存数据
      const {data:res} = await 
      this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
      {attr_name:row.attr_name,
      attr_sel:row.attr_sel,
      attr_vals:row.attr_vals.join(' ')
      })
      if(res.meta.status !== 200) return this.$message.error('修改参数项失败')
      this.$message.success('修改参数项成功')
    },
    // 删除对应的参数和选项
    handleClose(i,row){
      row.attr_vals.splice(i,1)
      this.saveAttrVals(row)
    }
  },
  computed: {
      isBtnDisabled(){
      if(this.selectedCateKeys.length !== 3) return true
      return false
    },
    cateId(){
      if(this.selectedCateKeys.length == 3) return this.selectedCateKeys[2]
      return null
    },
    titleText(){
      if(this.activeName === 'many'){
        return '动态参数'
      }
      else{
        return '静态属性'
      }
    },

  },
}
</script>

<style lang="less" scoped>
  .cat_opt{
    margin: 15px 0;
  }
  .el-tag{
    margin: 10px;
  }
  .input-new-tag{
    width: 120px;
  }
</style>