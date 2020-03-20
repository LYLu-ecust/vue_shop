<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-table-column>
          <el-button type="primary">添加角色</el-button>
        </el-table-column>
      </el-row>
      <el-table :data="roleList" stripe border>
        <el-table-column type="expand">
          <template v-slot="scope">
            <el-row :class="['bdbottom','vcenter',i1===0?'bdtop':'']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级权限 -->
              <el-col :span="19">
                <el-row :class="['vcenter',i2 === 0 ?'':'bdtop']" v-for="(item2,i2) in item1.children" :key="item2.id">
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
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width=300px>
          <template v-slot="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" >编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>

      </el-table>
    </el-card>
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="30%"
      @close = "setRightDialogClosed()">
      <!-- 树形控件 -->
      <el-tree 
      show-checkbox 
      :data="rightsList" 
      :props="treeProps" 
      :default-checked-keys="defKeys"
      node-key="id"
      default-expand-all
      ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRight()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data(){
    return{
      // 所有角色列表数据
      roleList:[],
      // 分配权限对话框
      setRightDialogVisible:false,
      // 所有权限数据
      rightsList:[],
      // 树形控件的属性绑定对象
      treeProps:{
        label:'authName',
        children:'children'
      },
      // 默认选定的节点id值数组
      defKeys:[],
      // 即将分配权限的角色id
      roleId:''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList(){
     const {data:res} = await this.$http.get('roles')
     if(res.meta.status !== 200) return this.$message.error("获取角色列表失败")
     this.roleList = res.data
     console.log(this.roleList)
    },
    // 根据is删除对应权限
    async removeRightById(role,rightId){
      console.log(role)
      // 弹框提示用户是否删除
      const confirmRes = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err =>err)
        if(confirmRes !== "confirm") return this.$message.info("取消了删除")
        const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if(res.meta.status !== 200) return this.$message.error("删除角色权限失败")
        role.children = res.data  
    },
    async showSetRightDialog(role){
      this.roleId = role.id
      const {data:res} = await this.$http.get('rights/tree')
      if(res.meta.status !== 200) return this.$message.error("获取用户权限列表失败")
      this.rightsList = res.data
      // 递归获取三级节点的id
      this.getLeafKeys(role,this.defKeys)
      console.log(this.rightsList)
      this.setRightDialogVisible=true
    },
    // 通过递归的形式获取所有三级id
    getLeafKeys(node,arr){
      // 当前节点不包含children属性则是三级节点
      if(!node.children){
        return arr.push(node.id)
      }
      node.children.forEach(item =>{
        this.getLeafKeys(item,arr)
      })
    },
    // 监听分配权限对话框关闭事件
    setRightDialogClosed(){
      this.defKeys = []
    },
    // 点击确定为角色分配权限
    async allotRight(){
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      console.log(idStr)
      const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
      console.log(res)
      if(res.meta.status !== 200) return this.$message.error("角色分配权限失败")
      this.$message.success("角色分配权限成功")
      this.getRolesList()
      this.setRightDialogVisible = false
      // console.log(keys)
    }

  },
}
</script>

<style lang="less" scoped>
  .el-tag{
    margin: 7px;
  }
  .bdtop{
    border-top: 1px solid #eee;
  }
  .bdbottom{
    border-bottom: 1px solid #eee;
  }
  .vcenter{
    display: flex;
    align-items: center;
  }
</style>