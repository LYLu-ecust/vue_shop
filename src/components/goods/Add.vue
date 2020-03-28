<template>
  <div>
        <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>

      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false">
      </el-alert>
      <!-- 步骤条区 -->
      <el-steps align-center :space="200" :active="activeIndex-0" finish-status="success">
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- tab栏区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
      <el-tabs 
      @tab-click = "tabClicked"
      :before-leave="beforeTabLeave" 
      v-model="activeIndex" 
      :tab-position="'left'">
        <el-tab-pane label="基本信息" name="0">
          <el-form-item label="商品名称" prop="goods_name"> 
            <el-input v-model="addForm.goods_name"></el-input>
          </el-form-item>
          <el-form-item label="商品价格" prop="goods_price"> 
            <el-input type="number" v-model="addForm.goods_price"></el-input>
          </el-form-item>
          <el-form-item label="商品重量" prop="goods_weight"> 
            <el-input type="number" v-model="addForm.goods_weight"></el-input>
          </el-form-item>
          <el-form-item label="商品数量" prop="goods_number"> 
            <el-input type="number" v-model="addForm.goods_number"></el-input>
          </el-form-item>
          <el-form-item label="商品分类" prop="goods_cat">
            <el-cascader
              v-model="addForm.goods_cat"
              :options="cateList"
              :props="cateProps"
              @change="handleChange"
              ></el-cascader>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品参数" name="1">
          <!-- 渲染表单的item项 -->
          <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
            <!-- 复选框组 -->
            <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox border :label="item" v-for="(item,index) in item.attr_vals" :key="index"></el-checkbox>
              </el-checkbox-group>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品属性" name="2">
          <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
            <el-input v-model="item.attr_vals"></el-input>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品图片" name="3">
          <!-- action表示图片要上传到的后台API地址，这个地址要写全 -->
          <el-upload
            :action="uploadURL"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            list-type="picture"
            :headers="headerObj"
            :on-success = "handleSuccess">
            <el-button size="small" type="primary">点击上传</el-button>
          </el-upload>
        </el-tab-pane>
        <el-tab-pane label="商品内容" name="4">
          <quill-editor
            v-model="addForm.goods_introduce"
          />
          <!-- 添加商品按钮 -->
          <el-button type="primary" class="btnAdd" @click="add">添加商品</el-button>
        </el-tab-pane>
      </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览对话框 -->
    <el-dialog
      title="图片预览"
      :visible.sync="previewVisible"
      width="50%">
      <img :src="previewPath" alt="" class="previewImg">   
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  data(){
    return{
      activeIndex:'0',
      // 添加商品的表单数据对象
      addForm:{
        goods_name:'',
        goods_price:0,
        goods_weight:0,
        goods_number:0,
        // 商品所属的分类数组
        goods_cat:[],
        // 图片数组
        pics:[],
        // 商品详情描述
        goods_introduce:'',
        attrs:[]
      },
      // 表单校验规则
      addFormRules:{
        goods_name:[
          {required:true,message:'请输入商品名称',trigger:'blur'}
        ],
        goods_price:[
          {required:true,message:'请输入商品价格',trigger:'blur'}
        ],
        goods_weight:[
          {required:true,message:'请输入商品重量',trigger:'blur'}
        ],
        goods_number:[
          {required:true,message:'请输入商品数量',trigger:'blur'}
        ],
        goods_cat:[
          {required:true,message:'请选择商品分类',trigger:'blur'}
        ]
      },
      // 商品分类列表
      cateList:[],
      // 级联选择器配置对象
      cateProps:{
        label:'cat_name',
        value:'cat_id',
        children:'children',
        expandTrigger: 'hover'
      },
      // 动态参数列表获取
      manyTableData:[],
      // 静态属性列表
      onlyTableData:[],
      // 上传文件的地址
      uploadURL:'http://127.0.0.1:8888/api/private/v1/upload',
      // 设置请求头
      headerObj:{Authorization:window.sessionStorage.getItem('token')},
      // 预览路径
      previewPath:'',
      previewVisible:false
    }
  },
  created() {
    this.getCateList()
  },
  methods:{
    async getCateList(){
      const {data:res} = await this.$http.get('categories')
      if(res.meta.status !== 200) return this.$message.error('获取商品分类数据失败')
      this.cateList = res.data
      // console.log(this.cateList)
    },
    // 级联选择器变化触发的函数
    handleChange(){
      console.log(this.addForm.goods_cat)
    },
    // 监听标签页离开事件,如果return false，就不能切换了
    // activeName为当前正要切换到的tab，oldActiveName为当前需要离开的
    beforeTabLeave(activeName,oldActiveName){
      // console.log(activeName + '<=' + oldActiveName)
      if(oldActiveName === '0' && this.addForm.goods_cat.length !== 3){
        this.$message.error('请选择商品分类')
        return false
      }
    },
    // 标签页点击切换
    async tabClicked(){
      if(this.activeIndex === '1'){
        const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:'many'}})
        if(res.meta.status !== 200) return this.$message.error('获取动态参数列表失败')
        res.data.forEach(item =>{
          item.attr_vals = item.attr_vals.length === 0 ? []:item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
        //  console.log(res.data)
      }
      else if(this.activeIndex === '2'){
        const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:'only'}})
        if(res.meta.status !== 200) return this.$message.error('获取静态属性列表失败')
        this.onlyTableData = res.data
        // console.log(res.data)
     }
    },
    // 处理图片预览
    handlePreview(file){
      this.previewPath = file.response.data.url
      this.previewVisible=true
      // console.log(file)
      // console.log(this.previewPath)
    },
    // 处理移出图片操作
    handleRemove(file){
      console.log(file)
      // 获取删除图片的临时路径
      const filePath = file.response.data.tmp_path
      // 根据临时路径查找到对应项索引值
      const i = this.addForm.pics.findIndex(x =>{
        x.pic === filePath
      })
      // 删除对应的图片
      this.addForm.pics.splice(i,1)
      console.log(this.addForm)
    },
    // 监听图片上传成功
    handleSuccess(response){
      // 拼接出图片对象，并将其push到数组中
      const picInfo = {pic:response.data.tmp_path}
      this.addForm.pics.push(picInfo)
      // console.log(this.addForm)
      // console.log(response)
    },
    // 添加商品
    add(){
      this.$refs.addFormRef.validate(async valid =>{
        if(!valid) return this.$message.error('请填写必要的表单项')
        // 用lodash的cloneDeep进行深拷贝
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数和静态属性
        this.manyTableData.forEach(item =>{
          const newInfo = {
            attr_id:item.attr_id,
            attr_value:item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        this.onlyTableData.forEach(item =>{
          const newInfo = {
            attr_id:item.attr_id,
            attr_value:item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        // 发起请求，添加商品
        // 商品名称必须是唯一的
        const {data:res} = await this.$http.post('goods',form)
        console.log(res)
        if(res.meta.status !== 201) return this.$message.error(res.meta.msg)
        this.$message.success('添加商品成功')
        this.$router.push('/goods')
      })
      // console.log(this.addForm)
    }
  },
  computed: {
    cateId(){
      if(this.addForm.goods_cat.length !== 3) return null
      return this.addForm.goods_cat[2]
    }
  },
}
</script>

<style lang="less" scoped>
  .el-steps{
    margin: 15px 0;
  }
  .el-step__title{
    font-size: 13px;
  }
  .el-checkbox{
    margin: 0 10px 0 0 !important;
  }
  .previewImg{
    width: 351px;
    // width: 100%;
    height: 269px;
  }
  .btnAdd{
    margin-top: 15px;
  }
</style>