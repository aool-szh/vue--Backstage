<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区 -->
    <el-card>
      <!-- 搜索和添加用户 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区 -->
      <el-table :data="userlist" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="姓名" width="180"></el-table-column>
        <el-table-column prop="email" label="邮箱" width="180"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
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
            <!-- 分配按钮 -->
            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页列表 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 添加用户额对话框 -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户对话框 -->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%">
      <el-form ref="editFormRef" :model="editForm" :rules="editFormRules" label-width="80px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 获取用户列表数据
      queryInfo: {
        query: '',
        // 当期的页数
        pagenum: 1,
        // 每页显示的数据条数
        pagesize: 10
      },
      userlist: [],
      total: 0,
      addDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addFormRules: {
        username: [{ required: true, message: '请输入用户名', trigger: 'blur' }, {
          min: 3, max: 10, message: '长度为3~5字符', trigger: 'blur'
        }],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }, {
          min: 3, max: 10, message: '长度为3~5字符', trigger: 'blur'
        }],
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' }, {
          min: 3, max: 20, message: '长度为3~5字符', trigger: 'blur'
        }],
        mobile: [{ required: true, message: '请输入电话', trigger: 'blur' }, {
          min: 3, max: 20, message: '长度为3~5字符', trigger: 'blur'
        }]
      },
      editFormRules: {
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' }, {
          min: 3, max: 20, message: '长度为3~5字符', trigger: 'blur'
        }],
        mobile: [{ required: true, message: '请输入电话', trigger: 'blur' }, {
          min: 3, max: 20, message: '长度为3~5字符', trigger: 'blur'
        }]
      },
      // 控制修改用户对话框的显示与隐藏
      editDialogVisible: false,
      editForm: {}
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    // 获取用户列表数据
    async getUserList() {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表shibai')
      }
      this.userlist = res.data.users
      this.total = res.data.total
    },
    // 监听Pagesize改变的事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听页码值改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听Switch状态的改变
    async userStateChanged(userinfo) {
      const { data: res } = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`
      )
      console.log(userinfo)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('更新失败')
      }
      this.$message.success('更新成功')
    },
    // 监听对话框关闭
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮，增加新用户
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addForm)
        console.log(res)
        if (res.meta.status !== 201) {
          this.$message.error('添加失败')
          return
        }
        this.$message.success('添加成功')
        this.addDialogVisible = false
        this.getUserList()
      }
      )
    },
    // 展示编辑用户的对话框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('users/' + id)

      if (res.meta.status !== 200) {
        return this.$message.error('查询失败')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 修改信息并提交
    editUserInfo() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('users/' + this.editForm.id, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        console.log(res)

        if (res.meta.status !== 200) {
          return this.$message.error('更新用户信息失败')
        }

        this.editDialogVisible = false

        this.getUserList()
        this.$message.success('更新用户信息成功')
      })
    },
    // 删除用户
    async removeUserById(id) {
      const confirmRsult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmRsult !== 'confirm') {
        return this.$message.info('已取消了删除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }

      this.$message.success('删除用户成功')
      this.getUserList()
    }

  }
}
</script>

<style>

</style>
