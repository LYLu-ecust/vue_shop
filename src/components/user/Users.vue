<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="9">
          <!-- 搜索框 -->
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button @click="getUserList" slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域绘制 -->
      <el-table :data="userList" stripe border>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态" >
          <template v-slot="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template v-slot="scope">
            <!-- 修改按钮 -->
            <el-button @click="showEditDialog(scope.row.id)" type="primary" icon="el-icon-edit" size="mini"></el-button>
            <!-- 删除按钮 -->
            <el-button @click="removeUserById(scope.row.id)" type="danger" icon="el-icon-delete" size="mini"></el-button>
            <!-- 分配角色 -->
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button @click="setRole(scope.row)" type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[1, 2, 5, 15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed">
      <!-- 内容主体区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
        <el-input v-model="addForm.username"></el-input>
      </el-form-item>
        <el-form-item label="密码" prop="password">
        <el-input v-model="addForm.password"></el-input>
      </el-form-item>
        <el-form-item label="邮箱" prop="email">
        <el-input v-model="addForm.email"></el-input>
      </el-form-item>
        <el-form-item label="手机" prop="mobile">
        <el-input v-model="addForm.mobile"></el-input>
      </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
<!-- 修改用户的对话框 -->
    <el-dialog @close="editDialogClosed()" title="修改用户" :visible.sync="editDialogVisible" width="50%">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
          <el-input disabled v-model="editForm.username"></el-input>
        </el-form-item> 
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item> 
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item> 
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo()">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 为用户分配角色对话框 -->
    <el-dialog
      title="分配角色"
      :visible.sync="setRoleDialogVisible"
      width="50%"
      @close = "setRoleDialogClosed()"
      >
      <div>
        <p>当前的用户：{{userinfo.username}}</p>
        <p>当前的角色：{{userinfo.role_name}}</p>
        <p>分配新的的角色：
          <el-select v-model="selectedRoleId" placeholder="请选择">
            <el-option
              v-for="item in rolesList"
              :key="item.id"
              :label="item.roleName"
              :value="item.id">
            </el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data(){
    // 邮箱验证
    var checkEmail = (rule,value,cb) =>{
      const regEmail =   /^([a-zA-Z]|[0-9])(\w|\-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/
      if(regEmail.test(value)){
        return cb()
      }
      cb(new Error("请输入合法的邮箱"))
    }
    // 验证手机号
    var checkMobile = (rule,value,cb) =>{
      const regMobile =   /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if(regMobile.test(value)){
        return cb()
      }
      cb(new Error("请输入合法的手机号"))
    }
    return{
      // 获取用户列表参数
      queryInfo:{
      query:"",
      // 当前页码
      pagenum:1,
      // 当前页多少条数据
      pagesize:2
      },
      userList:[],
      total:0,
      addDialogVisible:false,
      addForm:{},
      addFormRules:{
        username:[
          {required:true,message:'请输入用户名',trigger:'blur'},
          {min:3,max:10,message:'用户名长度在3~10个字符之间',trigger:'blur'}
        ],
        password:[
          {required:true,message:'请输入密码',trigger:'blur'},
          {min:6,max:15,message:'用户名长度在6~15个字符之间',trigger:'blur'}
        ],
        email:[
          {required:true,validator:checkEmail,trigger:'blur'}
        ],
        mobile:[
          {required:true,validator:checkMobile,trigger:'blur'}
        ]
      },
      editDialogVisible:false,
    //  查询到的用户信息赋值
     editForm:{},
    //  修改表单的验证规则
    editFormRules:{
      email:[
        {required:true,message:'请输入用户邮箱',trigger:'blur'},
        {validator:checkEmail,trigger:'blur'}
      ],
      mobile:[
        {required:true,message:'请输入用户手机',trigger:'blur'},
        {validator:checkMobile,trigger:'blur'}
      ]
    },
    setRoleDialogVisible:false,
    userinfo:{},
    // 所有角色列表
    rolesList:[],
    // 已选中的id值
    selectedRoleId:''
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList(){
      const {data:res} = await this.$http.get('users',{params:this.queryInfo})
      if(res.meta.status !== 200) return this.$message.error("获取用户列表失败")
      this.userList = res.data.users;
      this.total = res.data.total;
    },
    // 下拉框选择当页多少条数据
    handleSizeChange(newSize){
      this.queryInfo.pagesize = newSize;
      this.getUserList()
    },
    // 当前页数变化
    handleCurrentChange(newPage){
      this.queryInfo.pagenum = newPage;
      this.getUserList()
    },
    // 监听switch开关变化
    async userStateChanged(userinfo){
      const {data:res} = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if(res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error("更新用户失败")
      }
      this.$message.success("更新用户状态成功")
      // console.log(res)
    },
    // 监听添加用户对话框关闭
    addDialogClosed(){
      this.$refs.addFormRef.resetFields()
    },
    addUser(){
      this.$refs.addFormRef.validate(async valid =>{
        if(!valid) return
        const{data:res}=await this.$http.post('users',this.addForm)
        if(res.meta.status !== 201){
          this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        this.addDialogVisible = false
        // 重新获取用户列表
        this.getUserList()
      })
    },
    // 展示编辑用户对话框
    async showEditDialog(id){
      // console.log(id)
      const {data:res}=await this.$http.get('users/'+id)
      if(res.meta.status !== 200) return this.$message.error("用户查询失败")
      this.editForm = res.data;
      this.editDialogVisible = true
    },
    // 监听修改用户对话框关闭事件
    editDialogClosed(){
      this.$refs.editFormRef.resetFields();
    },
    editUserInfo(){
      this.$refs.editFormRef.validate(async valid =>{
        if(!valid) return
        const {data:res} = await this.$http.put('users/'+this.editForm.id,{
          email:this.editForm.email,
          mobile:this.editForm.mobile
        })
        if(res.meta.status !== 200) return this.$message.error("更新用户失败")
        // 关闭对话框
        this.editDialogVisible = false;
        // 刷新列表
        this.getUserList();
        // 提示更新成功
        this.$message.success("更新用户信息成功")
      })
    },
    async removeUserById(id){
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if(confirmResult !== 'confirm') return this.$message.info('已取消删除')
        // this.$message.success('以确认删除')
          const {data:res} = await this.$http.delete('users/'+id)
          if(res.meta.status !== 200) return this.$message.error('删除用户失败')
          this.$message.success('删除用户成功')
          this.getUserList()
        // console.log(confirmResult)
    },
    // 展示分配角色的对话框
    async setRole(userinfo){
      // 获取所有角色列表
      const{data:res} = await this.$http.get('roles')
      if(res.meta.status !== 200) this.$message.error("获取角色列表失败")
      this.rolesList = res.data
      this.userinfo = userinfo
      this.setRoleDialogVisible = true
    },
    // 保存用户分配的角色
    async saveRoleInfo(){
      if(!this.selectedRoleId) return this.$message.error("请选择要分配的角色")
      const {data:res} = await this.$http.put(`users/${this.userinfo.id}/role`,
      {
        rid:this.selectedRoleId
      })
      if(res.meta.status !== 200) return this.$message.error("更新角色失败")
      this.$message.success("更新角色成功")
      this.getUserList()
      this.setRoleDialogVisible = false
    },
    // 分配角色对话框清空事件
    setRoleDialogClosed(){
      this.selectedRoleId = ''
      this.userinfo = {}
    }
  },
}
</script>
<style lang="less" scoped>

</style>
