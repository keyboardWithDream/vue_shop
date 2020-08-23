<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 提示区域 -->
      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false"
      >
      </el-alert>
      <!-- 步骤区域 -->
      <el-steps
        :space="200"
        :active="activeIndex-0"
        finish-status="success"
        align-center
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="信息内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- 左侧标签页、表单 -->
      <el-form
        :model="addForm"
        :rules="addFormrules"
        ref="addFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-tabs
          :tab-position="'left'"
          v-model="activeIndex"
          :before-leave="beforeTabLeave"
          @tab-click="tabClicked"
        >
          <el-tab-pane
            label="基本信息"
            name="0"
          >
            <el-form-item
              label="商品名称"
              prop="goods_name"
            >
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item
              label="商品价格"
              prop="goods_price"
            >
              <el-input
                v-model="addForm.goods_price"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item
              label="商品数量"
              prop="goods_number"
            >
              <el-input
                v-model="addForm.goods_number"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item
              label="商品重量"
              prop="goods_weight"
            >
              <el-input
                v-model="addForm.goods_weight"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item
              label="商品分类"
              prop="goods_cat"
            >
              <!-- 级联选择器 -->
              <el-cascader
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="addProps"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>

          <el-tab-pane
            label="商品参数"
            name="1"
          >
            <!-- 商品参数表单项 -->
            <el-form-item
              :label="item.attr_name"
              v-for="item in manyTableData"
              :key="item.attr_id"
            >
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  border
                  :label="cb"
                  v-for="(cb,i) in item.attr_vals"
                  :key="i"
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane
            label="商品属性"
            name="2"
          >
            <!-- 商品属性表单项 -->
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTableData"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane
            label="商品图片"
            name="3"
          >
            <!-- 上传图片区域 -->
            <el-upload
              action="http://127.0.0.1:8888/api/private/v1/upload"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headersObj"
              :on-success="handelPicSuccess"
            >
              <el-button
                size="small"
                type="primary"
              >点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane
            label="商品内容"
            name="4"
          >
            <!-- 富文本区域 -->
            <quill-editor
              ref="myQuillEditor"
              v-model=" addForm.goods_introduce"
            />
            <el-button
              type="primary"
              @click="add"
            >添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览对话框 -->
    <el-dialog
      title="图片预览"
      :visible.sync="previewVisible"
      width="50%"
    >
      <img
        :src="previewPath"
        alt=""
        class="previewImg"
      >
    </el-dialog>
  </div>
</template>

<script>
//导入lodash用于深拷贝
import _ from 'lodash'

export default {
    data(){
        return{
          activeIndex: '0',
          //添加商品表单
          addForm:{
            goods_name:'',
            goods_price:0,
            goods_number:0,
            goods_weight:0,
            goods_cat:[],
            //保存商品图片的数组
            pics:[],
            //商品介绍
            goods_introduce:'',
            attrs:[]
          },
          //添加商品表单验证规则
          addFormrules:{
            goods_name:[{required:true,message:'请输入商品名称',trigger:'blur'}],
            goods_price:[{required:true,message:'请输入商品价格',trigger:'blur'}],
            goods_number:[{required:true,message:'请输入商品数量',trigger:'blur'}],
            goods_weight:[{required:true,message:'请输入商品重量',trigger:'blur'}],
            goods_cat:[{required:true,message:'请输入商品分类',trigger:'blur'}]
          },
          //保存分类数据列表
          cateList:[],
          //级联选择器配置项
          addProps:{ 
          expandTrigger: 'hover',
          label:'cat_name',
          value:'cat_id' },
          //保存商品参数数据
          manyTableData:[],
          //保存商品属性
          onlyTableData:[],
          //图片上传组件的headers对象
          headersObj:{
            Authorization : window.sessionStorage.getItem("token")
          },
          //图片预览地址
          previewPath:'',
          //图片预览对话框的显示和隐藏
          previewVisible:false,
         
        }
    },
    created(){
       this.getCateList()
    },
    methods:{
      async getCateList(){
        const {data:res}=await this.$http.get('categories')
        if(res.meta.status!==200){
          return this.$message.error('获取商品列表失败')
        }
        this.cateList=res.data
        // console.log(this.cateList); 
      },
      //级联选择框触发事件
      handleChange(){
        if(this.addForm.goods_cat.length!==3){
          this.addForm.goods_cat=[]
        }
      },
      //tab页切换的条件
      beforeTabLeave(activeName,oldActiveName){
       if(oldActiveName==='0'&&this.addForm.goods_cat.length!==3){
         this.$message.error('请选择三级商品分类列表')
         return false
       }
      },
      //切换到商品参数
     async tabClicked(){
        if(this.activeIndex==='1'){
          const {data:res}=await this.$http.get(`categories/${this.cateId}/attributes`,
          {  params:{sel:'many'}})
          if(res.meta.status!==200){
            return this.$message.error('获取商品参数失败')
          }
          res.data.forEach(item=>{
            item.attr_vals=
             item.attr_vals.length===0 
               ? [] 
               : item.attr_vals.split(' ')
          })
          this.manyTableData=res.data
          // console.log(this.manyTableData);
        }else if(this.activeIndex==='2'){
          const {data:res}=await this.$http.get(`categories/${this.cateId}/attributes`,
          {  params:{sel:'only'}})
           if(res.meta.status!==200){
            return this.$message.error('获取商品属性失败')
          }
          this.onlyTableData=res.data
          // console.log(this.onlyTableData);
        }
      },
      //处理图片预览效果
      handlePreview(file){
        this.previewPath=file.response.data.url
        this.previewVisible=true
      },
      //图片移除操作
      handleRemove(file){
        // console.log(file);
        //1.找到移除图片路径
        const filePath =file.response.data.tmp_path
        //2.移除图片索引
        const i = this.addForm.pics.findIndex(x=>{
          x.pic===filePath
        })
        //3.从数组中移除掉
        this.addForm.pics.splice(i,1)
        // console.log(this.addForm);
      },
      //图片上传成功事件
      handelPicSuccess(response){
        // console.log(response);
        //1.拼接上传成功的图片对象
        const picInfo = {pic:response.data.tmp_path}
        //2.把对象push到图片数组里
        this.addForm.pics.push(picInfo)
        // console.log(this.addForm);
      },
      //添加商品事件
      add(){
        this.$refs.addFormRef.validate(async valid=>{
          if(!valid){
           return this.$message.error('请填写必要的表单项')
          }
          //执行添加操做
          const form = _.cloneDeep(this.addForm)
          form.goods_cat=form.goods_cat.join(',')

          this.manyTableData.forEach(item=>{
            form.attrs.push({ attr_id:item.attr_id, attr_value:item.attr_vals.join(" ") }) 
          })
          //将onlyTableData（静态属性）处理添加到attrs
          this.onlyTableData.forEach(item=>{
            form.attrs.push({ attr_id:item.attr_id, attr_value:item.attr_vals }) 
          })
          console.log(form);
          
          
          // 发起添加商品请求
          const{data:res}=await this.$http.post('goods',form)
          if(res.meta.statu!==201){
           this.$message.error('添加商品失败')
          } 
          this.$message.success('添加商品成功')
          this.$router.push('/goods')

        })
      }
        
    },
    computed:{
      cateId(){
        if(this.addForm.goods_cat.length===3){
          return this.addForm.goods_cat[2]
        }
        return null
      }
    }

}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.previewImg {
  width: 100%;
}
.el-button {
  margin-top: 10px;
}
</style>