<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 角色列表区 -->
    <el-card>
      <el-button type="primary">添加角色</el-button>
      <el-table
        :data=roleList
        border
        stripe
      >
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope='scope'>

            <!-- 栅格-->
            <el-row
              v-for="(item1,i1) in scope.row.children"
              :key="item1.id"
              :class="['dbbottom',i1===0?'dbtop':'','vcenter']"
            >
              <!--一级权限  -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close=removeRightById(scope.row,item1.id)
                >{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>

              <!--  二三级权限  -->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限  -->
                <el-row
                  v-for="(item2,i2) in item1.children"
                  :key="item2.id"
                  :class="[i2===0?'':'dbtop','vcenter']"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close=removeRightById(scope.row,item2.id)
                    >{{item2.authName}} </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3) in item2.children"
                      :key="item3.id"
                      closable
                      @close=removeRightById(scope.row,item3.id)
                    >{{item3.authName}} </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column
          label="角色名称"
          prop="roleName"
        ></el-table-column>
        <el-table-column
          label="角色描述"
          prop="roleDesc"
        ></el-table-column>
        <el-table-column
          label="操作"
          width="300px"
        >
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
            >删除</el-button>
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
            >分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分配权限对话框 -->
      <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        width="50%"
        @close="setRightDialogClosed"
      >
        <!-- 树形组件
          show-checkbox:显示复选框
          :props=""数据绑定的字段
          node-key:设置选中节点对应的值
          default-expand-all:是否默认展开所有节点
          :default-checked-keys 设置默认选中项的数组
          ref:设置引用 -->
        <el-tree
          :data='rightsList'
          :props="treeProps"
          show-checkbox
          node-key="id"
          default-expand-all
          :default-checked-keys="defKeys"
          ref='treeRef'
        ></el-tree>
        <span
          slot="footer"
          class="dialog-footer"
        >
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button
            type="primary"
            @click="allotRights"
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
          //角色列表数据
            roleList:[],
            //控制分配权限对话框的显示和隐藏
            setRightDialogVisible:false,
            //权限树数据
            rightsLis:[],
            //树形控件的属性绑定对象
            treeProps:{
              //树形节点的文本展示
              label:'authName',
              //设置通过children属性展示子节点
              children:'children'
            },
            //树形节点的默认选中内容
            defKeys:[],
            //保存正在操作的角色id
            roleId:''
        }
    },
    created(){
        this.getRoleList()
    },
    methods:{
        async getRoleList(){
            const{data:res}=await this.$http.get('roles')

            //请求失败
            if(res.meta.status!==200) return this.$message.error('获取角色列表失败')
            //请求成功
            this.roleList=res.data
            // console.log(res.data);
        },
        //根据id删除对应权限
       async removeRightById(role,rightId){
             //弹窗提示用户是否要删除
            const confirmResult = await this.$confirm('请问是否要删除该权限','删除提示',{
                confirmButtonText:'确认删除',
                cancelButtonText:'取消',
                type:'warning'
            }).catch(err=>err)

            //如果用户点击确认，则confirmResult 为'confirm'
            //如果用户点击取消, 则confirmResult获取的就是catch的错误消息'cancel'
            if(confirmResult != "confirm"){
                return this.$message.info("已经取消删除")
             }

             //点击确认删除
             const {data:res}= await this.$http.delete(`roles/${role.id}/rights/${rightId}`)

             if(res.meta.status!==200) return this.$message.error('删除角色失败')

                 //无需再重新加载所有权限
                 // this.getRoleList();
                //只需要对现有的角色权限进行更新即可
                 role.children = res.data
 
        },

        //分配权限请求数据并显示对话框
        async showSetRightDialog(role){
          this.roleId=role.id
          this.setRightDialogVisible = true
          //获取数据
          const {data:res} = await this.$http.get('rights/tree')
          if(res.meta.status!==200) return this.$message.error('获取权限树失败')

          //如果状态正常，将请求的数据保存在data中
          this.rightsList=res.data

          //调用递归获取的三级节点
          this.getLeafKeys(role,this.defKeys)
        },

         //获取到当前角色的所有三级权限id并添加到defKeys中
         getLeafKeys(node,arr){
           //如果当前节点不包含children属性，则表示node为三级权限
           if(!node.children){
             return arr.push(node.id)
           }
           //递归调用
           node.children.forEach(item => this.getLeafKeys(item, arr))
         },

         //当用户关闭树形权限对话框的时候，清除掉所有选中状态
         setRightDialogClose(){
           this.defKeys=[]
         },

         //当用户在树形权限对话框中点击确定，将用户选择的权限发送请求进行更新
        async allotRights(){
              //获取所有选中及半选的内容
              const keys=[
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
              ]
               //将数组转换为 , 拼接的字符串
               const idStr=keys.join(',')

               //发送请求完成更新
               const {data:res}=await this.$http.post(`roles/${this.roleId}/rights`,{ rids:idStr })

               if(res.meta.status!==200)   return this.$message.error('分配权限失败')

                  this.$message.success("分配权限成功")
                  this.getRoleList();
                  //关闭对话框
                  this.setRightDialogVisible = false;


         }

         
         
         
    }

}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.dbtop {
  border-top: solid 1px #eee;
}
.dbbottom {
  border-bottom: solid 1px #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>