<template>
  <div>
    <!--导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: './' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>客户信息管理</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter = '20'>
        <el-col :span = '7'>
          <el-input placeholder="请输入搜索内容" v-model="queryInfo.query" clearable @change="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span = '4'>
          <el-button type = "primary" @click = "addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!--这里时显示用户信息的部分-->
      <el-table :data = 'userList' stripe> <!--绑定数据为，userList-->


        <!--由于要展示的信息过多，因此在这里设置了扩展列，-->
        <el-table-column type="expand">
          <template slot-scope="props">
            <el-form label-position="left" inline class="demo-table-expand">

              <el-form-item label="电话号码">
                <span>{{ props.row.tel }}</span>
              </el-form-item>

            </el-form>
          </template>
        </el-table-column>

        <!--索引列-->
        <el-table-column  type="index"></el-table-column>
        <!--这里我就写两个作为样例了，具体展示多少信息，可以自己设置，或者再一起讨论-->
        <el-table-column label = '用户编码' >
          <template slot-scope="scope">
            <span style="margin-left: 10px">{{ scope.row.userId }}</span>
          </template>
        </el-table-column>
        <el-table-column label = '用户名称'>
          <template slot-scope="scope">
            <span style="margin-left: 10px">{{ scope.row.username }}</span>
          </template>
        </el-table-column>

        <el-table-column label = '公司单位'>
          <template slot-scope="scope">
            <span style="margin-left: 10px">{{ scope.row.company }}</span>
          </template>
        </el-table-column>

        <!--这里是对用户信息的修改删除-->
        <el-table-column label = '操作'>
          <template slot-scope="scope">     <!--scope用来接收数据-->
            <!--修改-->

            <el-tooltip effect="dark" content="修改信息" placement="top" :enterable="false">
            <el-button type="primary" icon="el-icon-edit" @click="showEdit(scope.row.userId)"></el-button><!--传入参数为用户id-->
            </el-tooltip>

            <!--删除-->

            <el-tooltip effect="dark" content="删除用户" placement="top" :enterable="false" >
            <el-button type="danger" icon="el-icon-delete" @click="removeUser(scope.row.userId)"></el-button>
            </el-tooltip>

          </template>
        </el-table-column>
      </el-table>

      <!--分页区域-->
      <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="queryInfo.pageNum"
              :page-sizes="[1, 2, 5, 10]"
              :page-size="queryInfo.pageSize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="total">
      </el-pagination>
    </el-card>

    <!--添加用户对话框-->
    <el-dialog
            title = "添加用户"
            @close="addDialogClosed"
            :visible.sync = "addDialogVisible"
            width = "30%">
        <el-form ref="form" :model="form" label-width="80px" >

          <el-form-item label="用户编号">
            <el-input v-model="form.userId"></el-input>
          </el-form-item>

          <el-form-item label="用户名称">
            <el-input v-model="form.username"></el-input>
          </el-form-item>

          <el-form-item label="电话号码">
            <el-input v-model="form.tel"></el-input>
          </el-form-item>

          <el-form-item label="用户单位">
            <el-input v-model="form.company"></el-input>
          </el-form-item>

          <el-form-item>
            <el-button type="primary" @click="onSubmit">添加</el-button>
            <el-button @click="reset">重置</el-button>
          </el-form-item>


      </el-form>
      <span slot = "footer" class = "dialog-footer">
        <el-button type="primary" @click="addDialogVisible = false">关闭</el-button>
      </span>
    </el-dialog>

    <!--修改用户的对话框-->
    <el-dialog
            title="修改用户信息"
            :visible.sync="editDialogVisible"
            width="50%">

      <el-form :model = 'editForm'
               ref    = 'editFormRef'
               label-width = '70px'>
        <el-form-item label="用户id">
          <el-input v-model="editForm.userId" disabled>
          </el-input>
        </el-form-item>

        <el-form-item label="用户名">
          <el-input v-model="editForm.username">
          </el-input>
        </el-form-item>

        <el-form-item label="用户电话">
          <el-input v-model="editForm.tel" >
          </el-input>
        </el-form-item>

        <el-form-item label="用户单位">
          <el-input v-model="editForm.company" >
          </el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUpdate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: "UserManage",
    data() {
      return {
        //获取用户列表的参数对象
        queryInfo: {
          query: '',    //搜索用到的关键字
          pageNum: 1,   //当前页数
          pageSize: 2,  //每页显示多少条数据
        },

        //发送网络请求时，请直接把所有的用户信息都返回给我,
        //在添加用户，和修改用户信息时，我的想法是这样的，发送网络请求把要修改的内容传过去，
        //你那边更新数据库内容，然后动态把新的userList再发给我，以达到客户端这边的动态显示修改内容
        userList: [
          {
            userId: '加载中',
            username: '',
            tel: '',
            company: '',
          }
          // {
          //   userId: '007',
          //   username: 'kobe',
          //   tel: '119',
          //   company: 'facebook'
          // },
          // {
          //   userId: '009',
          //   username: 'jordan',
          //   tel: '3553',
          //   company: 'ibm'
          // }
        ],
        total: 0,

        addDialogVisible: false, //控制添加用户信息对话框的显示和隐藏

        form: {
          userId: '',
          username: '',
          tel: '',
          company:''
        },   //添加用户所需要填写的表单

        editDialogVisible: false,//控制修改用户信息对话框的显示和隐藏

        editForm: {},            //填写修改用户信息的表单，数据库请求到该id的信息后显示在对话框上，供管理员修改
      }
    },
    created() {//这里的vue实例创建，即进入UserManage时，就向服务器发送网络请求，获取用户列表
      this.getUserList()//该方法见methods
    },
    methods: {
      handleSizeChange(newSize) {
        console.log(newSize);
        this.queryInfo.pageSize = newSize
      },
      handleCurrentChange(newPage) {
        console.log(newPage);
        this.queryInfo.pageNum = newPage
      },
      //重置添加用户的表单信息
      reset() {
        this.$refs.form.resetFields()
      },
      //提交添加内容，
      // 这里我是直接把添加的用户信息直接append到userList中作为测试
      //具体实现，请更新到数据库
      async onSubmit() {
        console.log(this.form);

        // if (this.userList.find(u => u.userId === this.form.userId) !== undefined) {
        //   this.$message.error("用户ID重复")
        //   return
        // }

        //this.userList.push(this.form)

        const {data: {feedback:feedbackInfo}} =
            await this.$http.post('a.general', {type: "add_user", user: this.form});

        if(feedbackInfo.length === 0) {
          this.$message.success("成功添加")
        }else{
          this.$message.error(feedbackInfo)
        }

        this.getUserList()
      },

      //展示用户编辑的对话框，来进行修改用户信息，这里请发送请求获取用户id
      showEdit(id) {
        //这里是我没直接用data里面的userList进行测试
        let n;
        console.log(id);
        for (n in this.userList) {
          //如果点击edit按钮后，传入的id和userList某一行相同，就将改条记录给editForm
          if (this.userList[n].userId === id) {
            this.editForm = this.userList[n]
            console.log(this.userList[n])
          }
        }
        this.editDialogVisible = true
      },

      //监听修改对话框关闭事件，这里关闭后，提交，更新数据到数据库
      async editUpdate() {
        const {data: {feedback:feedbackInfo}} =
            await this.$http.post('a.general', {type: "edit_user", user: this.editForm});

        if(feedbackInfo.length === 0) {
          this.$message.success("成功修改")
        }else{
          this.$message.error(feedbackInfo)
        }

        this.editDialogVisible = false // 关闭对话框

        this.getUserList()
      },

      //监听添加用户对话框关闭事件
      addDialogClosed() {
        this.$refs.form.resetFields()
      },

      //根据id删除用户
      async removeUser(id) {
        const confirmResult = await this.$confirm('此操作将永久删除用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => {
          return err
        })

        console.log(confirmResult)
        //如果用户确认删除，返回字符串confirm
        //如果用户点击取消，返回字符串cancel
        if (confirmResult !== 'confirm') {
          return this.$message('已取消')
        }
        //这里是直接操纵userList作为测试
        //let n
        console.log(id)

        const {data: {feedback:feedbackInfo}} =
            await this.$http.post('a.general',{type:"delete_user",userId:id})

        if(feedbackInfo.length === 0) {
          this.$message.success("删除用户成功")
        }else{
          this.$message.error(feedbackInfo)
        }

        this.getUserList() //更新用户列表
      },


      //网络请求请获得用户记录在下面函数里实现，具体是实现对用户列表userList和数据库里面的数据进行双向绑定

      //从数据库获取记录result，并更新到userList，返回来的result是一个对象数组
      async getUserList() {
        console.log('正在获取用户信息')

        const {data: respondInfo} = await this.$http.post('a.general',{
          type:"get_users", queryInfo:this.queryInfo
        });

        this.userList = respondInfo.userList;

        console.log('获得用户信息' + this.userList.length);
      },

    }
  }
</script>

<style scoped>

</style>