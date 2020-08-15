<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 搜索与添加区域 -->

      <el-row :gutter="20">
        <el-col :span=8>
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getUserList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList"
            ></el-button>
          </el-input>

        </el-col>
        <el-col :span=4>
          <el-button
            type="primary"
            @click="addDialogVisible=true"
          >添加用户</el-button>
        </el-col>

      </el-row>

      <!-- 用户列表区 -->
      <el-table
        :data="userlist"
        border
        stripe
      >
        <el-table-column type="index"></el-table-column>
        <el-table-column
          label="姓名"
          prop="username"
        ></el-table-column>
        <el-table-column></el-table-column>
        <el-table-column
          label="邮箱"
          prop="email"
        ></el-table-column>
        <el-table-column
          label="电话"
          prop="mobile"
        ></el-table-column>
        <el-table-column
          label="角色"
          prop="role_name"
        ></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStateChanged(scope.row)"
            ></el-switch>
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="180px"
        >
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            ></el-button>
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeUserById(scope.row.id)"
            ></el-button>

            <el-tooltip
              effect="dark"
              content=分配角色
              placement="top"
              :enterable="false"
            >
              <!-- 分配角色 -->
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="mini"
                @click="setRole(scope.row)"
              ></el-button>
            </el-tooltip>

          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 3, 4]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- //添加用户的对对话框 -->
      <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed"
      >
        <!-- 内容主体区 -->
        <el-form
          :model="addForm"
          :rules="addFormRules"
          ref="addFormRef"
          label-width="70px"
        >
          <el-form-item
            label="用户名"
            prop="username"
          >
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item
            label="密码"
            prop="password"
          >
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item
            label="邮箱"
            prop="email"
          >
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item
            label="手机"
            prop="mobile"
          >
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区域 -->
        <span
          slot="footer"
          class="dialog-footer"
        >
          <el-button @click="addDialogVisible=false">取 消</el-button>
          <el-button
            type="primary"
            @click="addUser"
          >确 定</el-button>
        </span>
      </el-dialog>

      <!-- 修改用户的对话框 -->
      <el-dialog
        title="修改用户"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClosed"
      >
        <!-- 主体区 -->
        <el-form
          :model="editForm"
          :rules="editFormRules"
          ref="editFormRef"
          label-width="70px"
        >
          <el-form-item label="用户名">
            <el-input
              v-model="editForm.username"
              disabled
            ></el-input>
          </el-form-item>

          <el-form-item
            label="邮箱"
            prop="email"
          >
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item
            label="手机"
            prop="mobile"
          >
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>

        <!-- 底部区 -->
        <span
          slot="footer"
          class="dialog-footer"
        >
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button
            type="primary"
            @click="editUserInfo"
          >确 定</el-button>
        </span>
      </el-dialog>

      <!-- 分配角色对话框 -->
      <el-dialog
        title="分配角色"
        :visible.sync="setRoleDialogVisible"
        width="50%"
        @close="setRolesDialogClosed"
      >
        <div>
          <p>当前的用户:{{userInfo.username}}</p>
          <p>当前的角色:{{userInfo.role_name}}</p>
          <p>分配新角色:

            <!-- 角色选择下拉框
          v-model：设置用户选中角色之后的id绑定数据
          -->
            <el-select
              v-model="selectedRoleId"
              placeholder="请选择角色"
            >
              <!-- :label 显示文本，:value 选中值 -->
              <el-option
                v-for="item in rolesList"
                :key="item.id"
                :label="item.roleName"
                :value="item.id"
              >
              </el-option>
            </el-select>
          </p>
        </div>
        <span
          slot="footer"
          class="dialog-footer"
        >
          <el-button @click="setRoleDialogVisible = false">取 消</el-button>
          <el-button
            type="primary"
            @click="saveRoleInfo"
          >确 定</el-button>
        </span>
      </el-dialog>

    </el-card>
  </div>
</template>

<script>
export default {
    data(){
        //验证邮箱的规则
    var checkEmail=(rule,value,cb)=>{
         const regEmail = /^\w+@\w+(\.\w+)+$/
          
          if(regEmail.test(value)){
              return cb()
          }

          cb(new Error('请输入合法的邮箱'))
    }
    //验证手机号的规则
    var checkMobile=(rule,value,cb)=>{
          const regMobile = /^1[34578]\d{9}$/

           
          if(regMobile.test(value)){
              return cb()
          }

          cb(new Error('请输入合法的手机号'))
    }
     return{
         //获取用户列表参数
         queryInfo:{
              // 查询的条件
             query:'',
             //当前页数
             pagenum:1,
             //当前每页显示的页数
             pagesize:2
         },
         userlist:[],
         total:0,
         //控制添加用户对话框的显示和隐藏
        addDialogVisible:false,
        //添加用户的表单数据
        addForm:{
            username:'',
            password:'',
            email:'',
            mobile:''
        },
        //添加表单的验证规则
        addFormRules:{
            username:[
                {
                    required:true,message:'请输入用户名',trigger:'blur'
                },{
                    min:3,max:10,message:'用户名的长度在3到10个之间' ,trigget:'blur'
                }
            ],
                   
            password:[{ required:true,message:'请输入密码',trigger:'blur'},
            {   min:6,max:15,message:'密码的长度在6到15个之间' ,trigget:'blur'}],
           
           email:[{required:true,message:'请输入邮箱',trigger:'blur'},{
                validator:checkEmail,trigger:'blur'
            }],
            mobile:[{required:true,message:'请输入手机号码',trigger:'blur'},{
                  validator:checkMobile,trigger:'blur'
            }]
        },
        //修改用户对话框显示或隐藏
        editDialogVisible:false,

        //修改时查询到的用户对象
        editForm:{},
        //修改表单的验证对象
        editFormRules:{
             email:[{required:true,message:'请输入邮箱',trigger:'blur'},{
                validator:checkEmail,trigger:'blur'
            }],
             mobile:[{required:true,message:'请输入手机号码',trigger:'blur'},{
                  validator:checkMobile,trigger:'blur'
            }]

        },

        //控制添加权限对话框的显示和隐藏
        setRoleDialogVisible:false,
        //保存正在操作的那个用户信息
        userInfo:{},
        //保存所有的角色信息
        rolesList:[],
        //保存用户选中的角色id
       selectedRoleId:''

        

     }
    },
    created(){
     this.getUserList()
    },
    methods:{
       async getUserList(){
            const {data:res}=await this.$http.get('users',{params:this.queryInfo})
            if(res.meta.status!==200) return this.$message.console.error('获取用户列表失败');
            
            this.userlist=res.data.users;
            this.total=res.data.total
        },
        //监听pagesize改变的事件
        handleSizeChange(newSize){
            console.log(newSize);
            this.queryInfo.pagesize=newSize
            this.getUserList()
        },
        //监听页码值改变的事件
        handleCurrentChange(newPage){
            console.log(newPage);
            this.queryInfo.pagenum=newPage
            this.getUserList()

        },
        //监听switch开关状态的事件
        async userStateChanged(userinfo){
            // console.log(userinfo);
         const{data:res} = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
         if(res.meta.status!==200)  {
             userinfo.mg_state=!userinfo.mg_state
            return this.$message.error('更新用户状态失败')
         }

         this.$message.success('更新用户状态成功')
        },

        //重置用户添加表单
         addDialogClosed(){
             this.$refs.addFormRef.resetFields()
              
         },

         //点击按钮添加新用户
         addUser(){
             this.$refs.addFormRef.validate(async valid=>{
                if(!valid) return this.$message.error("请填写完整用户信息");

                 //可以发起添加用户请求
                const {data:res}=await this.$http.post('users',this.addForm)
                if(res.meta.state!==201){
                    this.$message.error('添加用户失败')
                }
                this.$message.success('添加用户成功')
                this.addDialogVisible=false
                //重新获取用户数据
                this.getUserList()
             })
         },

         //展示编辑用户的对话框
         async showEditDialog(id){
            // console.log(id);
          const {data:res}= await this.$http.get('users/'+id)
          if(res.meta.status!==200) return this.$message.error('修改用户失败')

            this.editForm=res.data

            this.editDialogVisible=true
         },
        
        //监听修改用户对话框的关闭事件
        editDialogClosed(){
            this.$refs.editFormRef.resetFields()
        },
        //修改用户信息并提交
        editUserInfo(){
              //用户点击修改表单中的确定按钮之后，验证表单数据
            this.$refs.editFormRef.validate(async valid=>{
                if(!valid) return this.$message.error('请填写完整用户信息')
                 //发送请求完成修改用户的操作
                 const {data:res}=await this.$http.put('users/'+this.editForm.id,this.editForm)

                 //修改失败
                 if(res.meta.status!==200) return this.$message.error('修改用户失败')
                 //修改成功
                 this.$message.success('修改用户成功')
                 //关闭对话框
                 this.editDialogVisible=false
                 //请求最新的数据
                 this.getUserList()

            })

        },

        //根据id删除用户
       async removeUserById(id){
            //弹出对话框询问是否删除用户
            const confirmResult = await this.$confirm('请问是否要永久删除改用户','删除提示',{
                confirmBottonText:'确认删除',
                cancelButtonText:'取消',
                type:'warning'
            }).catch(err=>err)
            //用户点击确认confirmResult为'confirm'，点击取消为catch错误提示'cancle'
            if(confirmResult!=='confirm') return this.$message.info('已经取消删除操作')

            //发送请求根据id完成删除操作
            const {data:res}=await this.$http.delete('users/'+id)

            //判断如果删除失败就提示
            if(res.meta.status!==200) return this.$message.error('删除用户失败')

            //删除成功的提示
            this.$message.success('删除用户成功')
            //重新请求最新数据
            this.getUserList()
            

        },

        //监听权限添加
        async setRole(userInfo){
          this.setRoleDialogVisible = true;
          //保存起来以供后续使用
          this.userInfo=userInfo

          //获取所有的角色信息，以备下拉列表使用
          //发送请求根据id完成删除操作
           const { data: res } = await this.$http.get('roles')
            //判断如果删除失败，就做提示
            if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')

            this.rolesList=res.data
            //展示分配角色对话框
            this.setRoleDialogVisible = true;
        },

        //点击按钮分配角色
        async saveRoleInfo(){
          //当用户点击确定按钮之后
         //判断用户是否选择了需要分配的角色
         if(!this.selectedRoleId){
           return this.$message.error('请选择需要分配的角色')
         }

          //发送请求完成分配角色的操作
          const {data:res} = await this.$http.put(`users/${this.userInfo.id}/role`,{rid:this.selectedRoleId})

          //判断如果删除失败，就做提示
          if (res.meta.status !== 200)
            return this.$message.error('分配角色失败')

          this.$message.success('分配角色成功')
          this.getUserList();
          //关闭对话框
          this.setRoleDialogVisible = false

        },

        //关闭对话框的时候，重置下拉框中的内容
        setRolesDialogClosed(){
          this.selectedRoleId=''
          this.userInfo={}
        }

    }

}
</script>

<style lang="less" scoped>
</style>>

