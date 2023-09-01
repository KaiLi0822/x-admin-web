<template>
  <div>
    <el-card id="search">
      <el-row>
        <el-col :span="20">
          <el-input
            v-model="searchModel.username"
            placeholder="Username"
            clearable
          ></el-input>
          <el-input
            v-model="searchModel.phone"
            placeholder="Password"
            clearable
          ></el-input>
          <el-button
            @click="getUserList"
            type="primary"
            round
            icon="el-icon-search"
            >Search</el-button
          >
        </el-col>
        <el-col :span="4" align="right">
          <el-button
            @click="openEditUI(null)"
            type="primary"
            icon="el-icon-plus"
            circle
          ></el-button>
        </el-col>
      </el-row>
    </el-card>
    <el-card>
      <el-table :data="userList" stripe style="width: 100%">
        <!-- (pageNo - 1) * pageSize + index -1 -->
        <el-table-column label="#" width="80">
          <template slot-scope="scope">
            {{
              (searchModel.pageNo - 1) * searchModel.pageSize + scope.$index + 1
            }}
          </template>
        </el-table-column>
        <el-table-column prop="id" label="id" width="180"> </el-table-column>
        <el-table-column prop="username" label="username" width="180">
        </el-table-column>
        <el-table-column prop="phone" label="phone" width="180">
        </el-table-column>
        <el-table-column prop="status" label="status" width="180">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.status == 1">normal</el-tag>
            <el-tag v-else type="danger">abnormal</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="email" label="email"> </el-table-column>
        <el-table-column label="action" width="180"> 
          <template slot-scope="scope">
            <el-button @click="openEditUI(scope.row.id)" type="primary" icon="el-icon-edit" size="mini" circle></el-button>
            <el-button @click="deleteUser(scope.row)" type="danger" icon="el-icon-delete" size="mini" circle></el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分页组件 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="searchModel.pageNo"
      :page-sizes="[5, 10, 20, 50]"
      :page-size="searchModel.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    >
    </el-pagination>
    <!-- 用户信息编辑对话框 -->
    <el-dialog
      @close="clearForm"
      :title="title"
      :visible.sync="dialogFormVisible"
    >
      <el-form :model="userForm" ref="userFormRef" :rules="rules">
        <el-form-item
          label="username"
          :label-width="formLabelWidth"
          prop="username"
        >
          <el-input v-model="userForm.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item v-if="userForm.id == null || userForm.id == undefined"
          label="password"
          :label-width="formLabelWidth"
          prop="password"
        >
          <el-input
            type="password"
            v-model="userForm.password"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item label="phone" :label-width="formLabelWidth">
          <el-input v-model="userForm.phone" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="status" :label-width="formLabelWidth">
          <el-switch
            v-model="userForm.status"
            :active-value="1"
            :inactive-value="0"
          >
          </el-switch>
        </el-form-item>
        <el-form-item label="email" :label-width="formLabelWidth" prop="email">
          <el-input v-model="userForm.email" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveUser">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import userApi from "@/api/userManage";
export default {
  data() {
    var checkEmail = (rule, value, callback) => {
      var reg =
        /^[a-zA-Z0-9_.-]+@[a-zA-Z0-9-]+(\.[a-zA-Z0-9-]+)*\.[a-zA-Z0-9]{2,6}$/;
      if (!reg.test(value)) {
        return callback(new Error("The Format of Email is Wrong"));
      }
      callback();
    };
    return {
      formLabelWidth: "130px",
      userForm: {},
      dialogFormVisible: false,
      title: "",
      total: 0,
      searchModel: {
        pageNo: 1,
        pageSize: 10,
      },
      userList: [],
      rules: {
        username: [
          { required: true, message: "Please Input Username", trigger: "blur" },
          {
            min: 3,
            max: 10,
            message: "Length should be 3 to 10",
            trigger: "blur",
          },
        ],
        password: [
          {
            required: true,
            message: "Please Input Initial Password",
            trigger: "blur",
          },
          {
            min: 6,
            max: 16,
            message: "Length should be 6 to 16",
            trigger: "blur",
          },
        ],
        email: [
          { required: true, message: "Please Input Email", trigger: "blur" },
          { validator: checkEmail, trigger: "blur" },
        ],
      },
    };
  },
  methods: {
    saveUser() {
      //trigger form validate
      this.$refs.userFormRef.validate((valid) => {
        if (valid) {
          //submit user info
          userApi.saveUser(this.userForm).then(response => {
            // success message
            this.$message({
              message: response.message,
              type: 'success'
            });
            // close dialog
            this.dialogFormVisible = false;
            // refresh form
            this.getUserList();
          });
        } else {
          console.log("error submit!!");
          return false;
        }
      }); 
    },
    clearForm() {
      this.userForm = {};
      this.$refs.userFormRef.clearValidate();
    },
    handleCurrentChange(pageNo) {
      this.searchModel.pageNo = pageNo;
      this.getUserList();
    },
    handleSizeChange(pageSize) {
      this.searchModel.pageSize = pageSize;
      this.getUserList();
    },
    getUserList() {
      userApi.getUserList(this.searchModel).then((response) => {
        this.userList = response.data.rows;
        this.total = response.data.total;
      });
    },
    openEditUI(id) {
      if (id == null){
        this.title = "Add User";
      }else{
        this.title = "Update User";
        //Get User Info by Id
        userApi.getUserById(id).then(response => {
          this.userForm = response.data;
        })
      }
     
      this.dialogFormVisible = true;
    },
    deleteUser(user) {
      this.$confirm(`Are you sure you want to delete user ${user.username}`, 'Warning', {
          confirmButtonText: 'OK',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(() => {
          userApi.deleteUserById(user.id).then(response => {
            this.$message({
            type: 'info',
            message: response.message
            });
          this.getUserList();  
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: 'Delete canceled'
          });          
        });
    },
  },
  created() {
    this.getUserList();
  },
};
</script>

<style>
#search .el-input {
  width: 200px;
  margin-right: 10px;
}
.el-dialog .el-input {
  width: 85%;
}
</style>