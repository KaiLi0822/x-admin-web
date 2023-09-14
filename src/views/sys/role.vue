<template>
  <div>
    <el-card id="search">
      <el-row>
        <el-col :span="20">
          <el-input
            v-model="searchModel.roleName"
            placeholder="Rolename"
            clearable
          />
          <el-button
            type="primary"
            round
            icon="el-icon-search"
            @click="getRoleList"
          >Search</el-button>
        </el-col>
        <el-col :span="4" align="right">
          <el-button
            type="primary"
            icon="el-icon-plus"
            circle
            @click="openEditUI(null)"
          />
        </el-col>
      </el-row>
    </el-card>
    <el-card>
      <el-table :data="roleList" stripe style="width: 100%">
        <!-- (pageNo - 1) * pageSize + index -1 -->
        <el-table-column label="#" width="80">
          <template slot-scope="scope">
            {{
              (searchModel.pageNo - 1) * searchModel.pageSize + scope.$index + 1
            }}
          </template>
        </el-table-column>
        <el-table-column prop="roleId" label="roleId" width="200" />
        <el-table-column prop="roleName" label="roleName" width="260" />
        <el-table-column prop="roleDesc" label="roleDesc" />
        <el-table-column label="action" width="180">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" circle @click="openEditUI(scope.row.roleId)" />
            <el-button type="danger" icon="el-icon-delete" size="mini" circle @click="deleteRole(scope.row)" />
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分页组件 -->
    <el-pagination
      :current-page="searchModel.pageNo"
      :page-sizes="[5, 10, 20, 50]"
      :page-size="searchModel.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
    <!-- 角色信息编辑对话框 -->
    <el-dialog
      :title="title"
      :visible.sync="dialogFormVisible"
      @close="clearForm"
    >
      <el-form ref="roleFormRef" :model="roleForm" :rules="rules">
        <el-form-item
          label="roleName"
          :label-width="formLabelWidth"
          prop="roleName"
        >
          <el-input v-model="roleForm.roleName" autocomplete="off" />
        </el-form-item>
        <el-form-item
          label="roleDesc"
          :label-width="formLabelWidth"
          prop="roleDesc"
        >
          <el-input v-model="roleForm.roleDesc" autocomplete="off" />
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRole">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import roleApi from '@/api/roleManage'
export default {
  data() {
    return {
      formLabelWidth: '130px',
      roleForm: {},
      dialogFormVisible: false,
      title: '',
      total: 0,
      searchModel: {
        pageNo: 1,
        pageSize: 10
      },
      roleList: [],
      rules: {
        roleName: [
          { required: true, message: 'Please Input rolename', trigger: 'blur' },
          {
            min: 2,
            max: 20,
            message: 'Length should be 2 to 20',
            trigger: 'blur'
          }
        ],
        roleDesc: [
          { required: true, message: 'Please Input roledesc', trigger: 'blur' },
          {
            min: 2,
            max: 20,
            message: 'Length should be 2 to 20',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    deleteRole(role) {
      this.$confirm(`Are you sure you want to delete role ${role.roleName}`, 'Warning', {
        confirmButtonText: 'OK',
        cancelButtonText: 'Cancel',
        type: 'warning'
      }).then(() => {
        roleApi.deleteRoleById(role.roleId).then(response => {
          this.$message({
            type: 'info',
            message: response.message
          })
          this.getRoleList()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: 'Delete canceled'
        })
      })
    },
    saveRole() {
      // trigger form validate
      this.$refs.roleFormRef.validate((valid) => {
        if (valid) {
          // submit role info
          roleApi.saveRole(this.roleForm).then(response => {
            // success message
            this.$message({
              message: response.message,
              type: 'success'
            })
            // close dialog
            this.dialogFormVisible = false
            // refresh form
            this.getRoleList()
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    clearForm() {
      this.roleForm = {}
      this.$refs.roleFormRef.clearValidate()
    },
    handleCurrentChange(pageNo) {
      this.searchModel.pageNo = pageNo
      this.getRoleList()
    },
    handleSizeChange(pageSize) {
      this.searchModel.pageSize = pageSize
      this.getRoleList()
    },
    getRoleList() {
      roleApi.getRoleList(this.searchModel).then((response) => {
        this.roleList = response.data.rows
        this.total = response.data.total
      })
    },
    openEditUI(id) {
      if (id == null) {
        this.title = 'Add Role'
      } else {
        this.title = 'Update Role'
        // Get Role Info by Id
        roleApi.getRoleById(id).then(response => {
          this.roleForm = response.data
        })
      }

      this.dialogFormVisible = true
    }

  }
}
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
