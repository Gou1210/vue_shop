<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb>
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item><a href="/">用户管理</a></el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            class="input-with-select"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userlist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态" prop="mg_state">
          <template slot-scope="scope">
            <!-- scope:显示自定义列的内容，参数分别为row，column,$index -->
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStateChanged(scope.row)"
            ></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
              @click="showEditDialog(scope.row.id)"
            ></el-button>
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              size="mini"
              icon="el-icon-delete"
              @click="removeUserById(scope.row.id)"
            ></el-button>
            <!-- 分配角色按钮 -->
            <el-tooltip
              class="item"
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="warning"
                size="mini"
                icon="el-icon-setting"
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
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- 添加用户的对话框 -->
      <el-dialog
        title="添加用户"
        :visible.sync="addDialogVisible"
        width="30%"
        @close="addDialogClosed"
      >
        <!-- 内容主题区域 -->
        <el-form
          ref="addFormRef"
          :model="addForm"
          :rules="addFormRules"
          label-width="80px"
        >
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
              <!-- 修改用户对话框 -->
        <el-dialog
          title="提示"
          :visible.sync="editDialogVisible"
          width="30%"
          @close="editDialogClosed"
        >
                  <!-- 内容主题区域 -->
        <el-form
          
          :model="editForm"
                    ref="editFormRef"
          :rules="editFormRules"
          label-width="80px"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="editForm.username" disabled ></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>

        </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUserInfo(editForm.id)" 
              >确 定</el-button
            >
          </span>
        </el-dialog>
       

   
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    //   验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      // 验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-z0-9_-])/;
      if (regEmail.test(value)) {
        // 合法的邮箱
        return cb();
      }
      cb(new Error("请输入合法的邮箱"));
    };
    // 验证手机的规则
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/;
      if (regMobile.test(value)) {
        // 合法的手机号
        return cb();
      }
      cb(new Error("请输入合法的手机号"));
    };
    return {
      queryInfo: {
        query: "",
        //    当前页数
        pagenum: 1,
        //    当前每页显示多少条
        pagesize: 2,
      },
      userlist: [],
      total: 1,
      //    对话框显示与隐藏
      addDialogVisible: false,
    //   修改用户对话框的隐藏于显示
    editDialogVisible:false,
    // 删除用户对话框是否可见
    deleteDialogVisible:false,
    // 修改表单数据
    editForm:{},

      //    添加表单数据
      addForm: {
        username: "",
        password: "",
        email: "",
        mobile: "",
      },
      //    添加表单验证规则
      addFormRules: {
        username: [
          {
            required: true,
            message: "请输入正确用户名称",
            trigger: "blur",
          },
          {
            min: 3,
            max: 10,
            message: "用户名的长度在3-10个字之间",
            trigger: "blur",
          },
        ],
        password: [
          {
            required: true,
            message: "请输入正确密码格式",
            trigger: "blur",
          },
          {
            min: 3,
            max: 10,
            message: "用户名的长度在3-10个字之间",
            trigger: "blur",
          },
        ],
        email: [
          {
            required: true,
            message: "请输入正确邮箱格式",
            trigger: "blur",
          },
          { validator: checkEmail, trigger: "blur" },
          {
            min: 3,
            max: 30,
            message: "用户名的长度在3-30个字之间",
            trigger: "blur",
          },
        ],
        mobile: [
          {
            required: true,
            message: "请输入正确手机号码",
            trigger: "blur",
          },
          { validator: checkMobile, trigger: "blur" },
          {
            min: 1,
            max: 11,
            message: "用户名的长度在3-10个字之间",
            trigger: "blur",
          },
        ],
      },
      //    修改表单验证规则
      editFormRules: {
        username: [
          {
            required: true,
            message: "请输入正确用户名称",
            trigger: "blur",
          },
          {
            min: 3,
            max: 10,
            message: "用户名的长度在3-10个字之间",
            trigger: "blur",
          },
        ],

        email: [
          {
            required: true,
            message: "请输入正确邮箱格式",
            trigger: "blur",
          },
          { validator: checkEmail, trigger: "blur" },
          {
            min: 3,
            max: 30,
            message: "用户名的长度在3-30个字之间",
            trigger: "blur",
          },
        ],
        mobile: [
          {
            required: true,
            message: "请输入正确手机号码",
            trigger: "blur",
          },
          { validator: checkMobile, trigger: "blur" },
          {
            min: 1,
            max: 11,
            message: "用户名的长度在3-10个字之间",
            trigger: "blur",
          },
        ],
      },
    };
  },
  created() {
    this.getUserList();
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get("users", {
        params: this.queryInfo,
      });
      this.userlist = res.data.users;
      this.total = res.data.total;
    },
    handleSizeChange(e) {
      this.queryInfo.pagesize = e;
      this.getUserList();
    },
    handleCurrentChange(e) {
      this.queryInfo.pagenum = e;
      this.getUserList();
    },
    async userStateChanged(e) {
      const { data: res } = await this.$http.put(
        `users/${e.id}/state/${e.mg_state}`
      );
      if (res.meta.status !== 200) {
        e.mg_state = !e.mg_state;
        return;
      }
      this.$message.success("更新用户状态成功");
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields();
    },
    addUser() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.post("users", this.addForm);
        this.$message.success("添加用户成功");
        this.addDialogVisible = false;
        this.getUserList();
      });
    },
      async showEditDialog(id){
      
    const {data:res} = await this.$http.get('users/'+id)
    this.editForm = res.data
    console.log(this.editForm)
    this.editDialogVisible = true
  },
  editDialogClosed(){
      this.$refs.editFormRef.resetFields()
  },
  editUserInfo(id){
            this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.put('users/'+id, this.editForm);
        this.$message.success("修改用户成功");
        this.editDialogVisible = false;
        this.getUserList();
      });
  },
async removeUserById(id){
    const confirlResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err =>{
            return err
        }) 
         if(confirlResult!="cancel"){
         const { data: res } = await this.$http.delete('users/'+id);
        this.$message.success("删除用户成功");
        // this.deleteDialogVisible = false;
        this.getUserList();
        }
               

}

  },

};
</script>

<style lang="less" scoped>
.el-breadcrumb {
  margin-bottom: 15px;
}
</style>