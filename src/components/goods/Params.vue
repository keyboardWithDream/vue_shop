<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 警告区域 
      :closable="false"(是否展示“X”号)
       show-icon(显示图标) -->
      <el-alert
        title="注意：只允许为第三级分类设置相关参数"
        type="warning"
        :closable="false"
        show-icon
      >
      </el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!--级联选择器
              expandTrigger='hover'(鼠标悬停触发级联)
              v-model(设置级联菜单绑定数据)
              :options(指定级联菜单数据源)  
              :props(用来配置数据显示的规则) 
              clearable(提供“X”号完成删除文本功能) 
              checkStrictly:true(是否可以选中任意一级的菜单) -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cateProps"
            @change="handleChange"
          ></el-cascader>

        </el-col>
        <el-col>
          <!-- tab页签区域 -->
          <el-tabs
            v-model="activeName"
            @tab-click="handleTabClick"
          >
            <el-tab-pane
              label="动态参数"
              name="many"
            >

              <el-button
                size="mini"
                type="primary"
                :disabled="isButtonDisabled"
                @click="showAddDialogVisible"
              >添加参数</el-button>

              <!-- 动态参数表格 -->
              <el-table
                :data="manyTableData"
                border
                stripe
              >
                <!-- 展开行 -->
                <el-table-column type="expand">
                  <template slot-scope="scope">
                    <el-tag
                      v-for="(item,i) in scope.row.attr_vals"
                      :key="i"
                      closable
                      @close="handleClosed(i,scope.row)"
                    >
                      {{item}}
                    </el-tag>
                    <!-- 添加tag的输入框 -->
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
                    <el-button
                      v-else
                      class="button-new-tag"
                      size="small"
                      @click="showInput(scope.row)"
                    >+ New Tag</el-button>

                  </template>
                </el-table-column>
                <!-- 索引列 -->
                <el-table-column type="index"></el-table-column>
                <el-table-column
                  label="参数名称"
                  prop="attr_name"
                ></el-table-column>
                <el-table-column label="操作">
                  <template slot-scope="scope">
                    <el-button
                      size="mini"
                      type="primary"
                      icon="el-icon-edit"
                      @click="showEditDialog(scope.row.attr_id)"
                    >编辑</el-button>
                    <el-button
                      size="mini"
                      type="danger"
                      icon="el-icon-delete"
                      @click="removeParams(scope.row.attr_id)"
                    >删除</el-button>
                  </template>
                </el-table-column>
              </el-table>

            </el-tab-pane>

            <el-tab-pane
              label="静态参数"
              name="only"
            >

              <el-button
                size="mini"
                type="primary"
                :disabled="isButtonDisabled"
                @click="showAddDialogVisible"
              >添加属性</el-button>

              <!-- 静态属性表格 -->
              <el-table
                :data="onlyTableData"
                border
                stripe
              >
                <!-- 展开行 -->
                <el-table-column type="expand">
                  <template slot-scope="scope">
                    <el-tag
                      v-for="(item,i) in scope.row.attr_vals"
                      :key="i"
                      closable
                      @close="handleClosed(i,scope.row)"
                    >
                      {{item}}
                    </el-tag>
                    <!-- 添加tag的输入框 -->
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
                    <el-button
                      v-else
                      class="button-new-tag"
                      size="small"
                      @click="showInput(scope.row)"
                    >+ New Tag</el-button>

                  </template>
                </el-table-column>
                <!-- 索引列 -->
                <el-table-column type="index"></el-table-column>
                <el-table-column
                  label="属性名称"
                  prop="attr_name"
                ></el-table-column>
                <el-table-column label="操作">
                  <template slot-scope="scope">
                    <el-button
                      size="mini"
                      type="primary"
                      icon="el-icon-edit"
                    >编辑</el-button>
                    <el-button
                      size="mini"
                      type="danger"
                      icon="el-icon-delete"
                      @click="removeParams(scope.row.attr_id)"
                    >删除</el-button>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
          </el-tabs>
        </el-col>
      </el-row>

      <!-- 修改参数或属性对话框 -->
      <el-dialog
        title="修改"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClosed"
      >
        <!-- 添加表单 -->
        <el-form
          :model="editForm"
          :rules="editFormRules"
          ref="editFormRef"
          label-width="100px"
        >
          <el-form-item
            :label="titleText"
            prop="attr_name"
          >
            <el-input v-model="editForm.attr_name"></el-input>
          </el-form-item>
        </el-form>

        <span
          slot="footer"
          class="dialog-footer"
        >
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button
            type="primary"
            @click="editParams"
          >确 定</el-button>
        </span>
      </el-dialog>

      <!-- 添加参数对话框 -->
      <el-dialog
        :title="'添加'+titleText"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed"
      >
        <!-- 添加表单 -->
        <el-form
          :model="addForm"
          :rules="addFormRules"
          ref="addFormRef"
          label-width="100px"
        >
          <el-form-item
            :label="titleText"
            prop="attr_name"
          >
            <el-input v-model="addForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span
          slot="footer"
          class="dialog-footer"
        >
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button
            type="primary"
            @click="addParams"
          >确 定</el-button>
        </span>
      </el-dialog>

    </el-card>
  </div>
</template>

<script>
export default {
    data(){
        return{
            //分类列表
            cateList:[],
            //用户在级联下拉菜单中选中的分类id
            selectedCateKeys:[],
            //级联菜单展示规则
            cateProps:{ 
                value:'cat_id',
                label:'cat_name',
                children:'children',
                expandTrigger: 'hover' ,
            },
            //tab页签激活显示页签项
            activeName:'many',

            //用来保存动态数据
            manyTableData:[],
            //用来保存静态属性数据
            onlyTableData: [] ,

            //控制修改参数.属性对话框的显示或隐藏
            editDialogVisible:false,
            //修改参数.属性对话框中的表单
            editForm:{
                attr_name:'',
            },
            //修改表单的验证规则
             editFormRules:{
            attr_name:[
            { required: true, message: '请输入名称', trigger: 'blur' }
            ]
        },

         //添加对话框的显示和隐藏
         addDialogVisible:false,
         //添加参数表单
         addForm:{
             attr_name:''
         },
         //添加参数的验证规则
         addFormRules:{
             attr_name:[
            { required: true, message: '请输入名称', trigger: 'blur' }
            ]
         },



        }
    },
    created(){
        this.getCateList()
    },
    methods:{
        async getCateList(){
            const {data:res}=await this.$http.get('categories')
            if(res.meta.status!==200){
                return this.$message.error('获取分类数据失败')
            }
            //将数据列表赋值给cateList
            this.cateList=res.data
        },


        //当用户在级联菜单中选择内容改变时触发
       handleChange(){
           this.getParamsData()
           
        },
        handleTabClick() {
        console.log(this.activeName)
        this.getParamsData()
        },
        //获取参数列表数据
        async getParamsData(){
            //当选中的不是三级分类就把表格和列表清空
            if(this.selectedCateKeys.length!==3){
                this.selectedCateKeys=[]
                this.manyTableData=[]
                this.onlyTableData=[]
                return
            }
             // console.log(this.selectedCateKeys);
            // 发送请求根据用户选择的三级分类和面板获取参数数据
            const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,
            {params:{sel:this.activeName}}
            )
            if (res.meta.status !== 200) {
                return this.$message.error('获取参数列表数据失败')
            }

            //将获取到的数据中的attr_vals字符串转换为数组
            res.data.forEach(item=>{
                item.attr_vals=item.attr_vals?item.attr_vals.split(' '):[]
                 // 添加tag的文本框
                item.inputVisible=false
                //文本框里的值
                item.inputValue=''
               })
               
             // console.log(res.data)


            if (this.activeName === 'many') {
            //获取的是动态参数
            this.manyTableData = res.data
        } else if (this.activeName === 'only') {
            //获取的是静态属性
            this.onlyTableData = res.data
      }
        },
        

    //发起请求获取需要修改的那个参数数据
    async showEditDialog(attr_id){
        const {data:res}=await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`,
      {params:{ attr_sel:this.activeName }})
        if (res.meta.status !== 200) {
        return this.$message.error('获取参数数据失败')
      }
      this.editForm = res.data;
      //显示修改参数.属性对话框
      this.editDialogVisible = true;
        },

    //当关闭修改参数.属性对话框时
    editDialogClosed(){
        this.$refs.editFormRef.resetFields()
    },

    //当点击确认就验证表单，发送请求完成修改
     editParams(){
         this.$refs.editFormRef.validate(async valid=>{
             if(!valid) return;

        const {data:res} = await this.$http.put(
        `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
        {attr_name:this.editForm.attr_name,attr_sel:this.activeName})

        if (res.meta.status !== 200) {
          return this.$message.error('获取参数数据失败')
        }

        this.$message.success('修改' + this.titleText + '数据成功')
        this.editDialogVisible = false
        this.handleChange();
         })
     },

     //根据id删除对应的参数或属性事件
     async removeParams(attr_id){
         const confirmResult = await this.$confirm(
            '请问是否要删除该'+this.titleText,
            '删除提示',
            {
            confirmButtonText: '确认删除',
            cancelButtonText: '取消',
            type: 'warning'
            }
         ).catch(err=>err)
           //如果用户点击确认，则confirmResult 为'confirm'
          //如果用户点击取消, 则confirmResult获取的就是catch的错误消息'cancel'
          if(confirmResult!='confirm'){
              return this.$message.info('已经取消删除了')
          }

          //没有取消就是要删除，发送请求完成删除
          const {data:res}=await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)

             if (res.meta.status !== 200) {
                return this.$message.error('删除参数数据失败')
            }
            this.$message.success('删除' + this.titleText + '数据成功')
            this.handleChange()           
     },

        //控制添加参数对话框的显示和隐藏
        showAddDialogVisible(){
            this.addDialogVisible=true
        },
        //添加对话框的关闭事件
        addDialogClosed(){
            this.$refs.addFormRef.resetFields()
        },
        //点击确定添加参数，调用接口
        addParams(){
            this.$refs.addFormRef.validate(async valid=>{
                if(!valid) return;

                const{data:res}=await this.$http.post(`categories/${this.cateId}/attributes`,{
                    attr_name:this.addForm.attr_name,
                    attr_sel:this.activeName
                })
                if(res.meta.status!==201){
                    return this.$message.error('添加参数失败')
                }
                this.$message.success('添加参数成功')
                this.addDialogVisible=false
                this.getParamsData()
            })
        },

        //输入框失去焦点或按下enter触发的事件
       async handleInputConfirm(row){
            if(row.inputValue.trim().length===0){
                row.inputValue=''
                row.inputVisible=false
                return
            }
            //如果没有return则说明需要进行后续处理
            //把输入的值添加到数组并清空、隐藏
            row.attr_vals.push(row.inputValue)
             row.inputValue=''
             row.inputVisible=false
            //发送请求把添加的内容更新到数据库
            this.saveAttrVals(row)         
        },
        //添加或删除参数项数据库更新
        async saveAttrVals(row){
         const {data:res}=await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
            {attr_name:row.attr_name,
             attr_sel:row.attr_sel,
             attr_vals:row.attr_vals.join(' ')})
             
            if(res.meta.status!==200){
             return this.$message.error('修改参数项失败')
            }   
            this.$message.success('修改参数项成功！')

        },

        //删除参数可选项
        handleClosed(i,row){
            row.attr_vals.splice(i,1)
            this.saveAttrVals(row)
        },

        //点击按钮显示输入框
        showInput(row){
            row.inputVisible=true
          //$nextTick方法的做用是：页面上的元素被重新渲染之后才会调用回调函数里的代码
          this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
        }

        
    },
    computed:{
        // 按钮禁用与否
        isButtonDisabled(){
            return this.selectedCateKeys.length!==3
        },
        //获取选中三级分类id
        cateId(){   
          if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[this.selectedCateKeys.length - 1]
      }
      return null
        },
        //动态参数和静态参数切换
        titleText(){
            if(this.activeName==='many') {
                return '动态参数'
            }
            return '静态参数'
        }
    }

}
</script>

<style lang="less" scoped>
</style>