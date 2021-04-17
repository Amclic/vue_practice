<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <!-- 搜索添加 -->
            <el-row :gutter="20">
                <el-col :span="8">
                        <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="gitUserList">
                            <el-button slot="append" icon="el-icon-search" @click="gitUserList"></el-button>
                        </el-input>
                </el-col>
                <el-col :span="6">
                    <el-button type='primary' @click="addDilogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>

            <!-- 用户列表 -->
            <el-table
            :data="userlist"
            border
            stripe>
                <el-table-column type="index"></el-table-column>
                <el-table-column
                    prop="username"
                    label="姓名">
                </el-table-column>
                <el-table-column
                    prop="email"
                    label="邮箱">
                </el-table-column>
                <el-table-column
                    prop="mobile"
                    label="电话">
                </el-table-column>
                <el-table-column
                    prop="role_name"
                    label="角色">
                </el-table-column>
                <el-table-column
                    label="状态">
                    <!-- 用slot-scope="scope"接受作用域数据 .row是该行的 作用域插槽会覆盖prop -->
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChaner(scope.row)">
                        </el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="180px">
                    <template slot-scope="scope">
                        <!-- 修改按钮 -->
                        <el-button @click="showEditDilog(scope.row.id)" type="primary" icon="el-icon-edit" size="mini"></el-button>
                        <!-- 删除按钮 -->
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
                        <!-- 分配角色按钮 -->
                        <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>

            <!-- 分页区 -->
             <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[1, 2, 5, 10]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
            </el-pagination>
        </el-card>

        <!-- 添加用户 -->
        <el-dialog
        title="添加用户"
        :visible.sync="addDilogVisible"
        width="50%"
        @close="addDilogClosed">
            <!-- 内容区 -->
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
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="addForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部区 -->
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDilogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 修改用户信息 -->
        <el-dialog
        title="修改用户信息"
        :visible.sync="editDilogVisible"
        width="50%"
        @close="editDilogClosed">
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
                <el-form-item label="用户名">
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
            </el-form>
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
            </el-form>
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
                <el-form-item label="手机" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>
 
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDilogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editUserInfo">确 定</el-button>
            </span>
        </el-dialog>

        <!-- 分配角色对话框 -->
        <el-dialog
          title="分配角色"
          :visible.sync="setRoleDialogVisible"
          width="50%"
          @close="setRoleDialogClosed">
          <div>
            <p>当前用户:{{userInfo.username}}</p>
            <p>当前角色:{{userInfo.role_name}}</p>
            <p>分配新角色:
              <el-select v-model="selectedRoleId" placeholder="请选择">
                <el-option
                  v-for="item in rolesList"
                  :key="item.id"
                  :label="item.roleName"
                  :value="item.id">
                </el-option>
              </el-select>
            </p>
          </div>
          <span slot="footer" class="dialog-footer">
            <el-button @click="setRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="saveRoleInfo()">确 定</el-button>
          </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
  data() {
    var checkEmail = (rule, value, callback) => {
      // 验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        // 合法的邮箱
        return callback()
      }
      callback(new Error('请输入合法的邮箱'))
    }
    var regMobile = (rule, value, callback) => {
      // 验证手机号的正则表达式
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的手机号'))
    }

    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userlist: [],
      total: 0,
      // 隐藏对话框
      addDilogVisible: false,
      editDilogVisible: false,
      // 添加表单的数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表单的验证规则
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: regMobile, trigger: 'blur' }
        ]
      },
      // 查询到的用户数据
      editForm: {},
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: regMobile, trigger: 'blur' }
        ]   
      },
      // 控制分配角色的对话框显示
      setRoleDialogVisible: false,
      userInfo: {},
      // 所有角色的数据列表
      rolesList: [],
      // 已选中角色的id值
      selectedRoleId: ''
    }
  },
  created() {
    this.gitUserList()
  },
  methods: {
    async gitUserList() {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.userlist = res.data.users
      this.total = res.data.total
    //   console.log(res.data)
    },
    // 监听页面改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.gitUserList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.gitUserList()
    },
    // 监听开关的状态改变
    async userStateChaner(userinfo) {
      const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      // console.log(res)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.meta.status
        return this.$message.err('更新用户状态失败')
      }
      this.$message.success('更新用户状态成功')
    },
    addDilogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 添加用户验证
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败')
        }
        this.$message.success('添加用户成功')
        // 隐藏添加的对话框
        this.addDilogVisible = false
        // 重新获取列表
        this.gitUserList()
      })
    },
    // 编辑用户信息
    async showEditDilog(id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户数据失败')
      }
      this.editForm = res.data

      this.editDilogVisible = true
    },
    editDilogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 修改用户信息
    editUserInfo() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('users/' + this.editForm.id,
          {
            email: this.editForm.email,
            mobile: this.editForm.mobile
          })
        
        if (res.meta.status !== 200) {
          this.$message.error('更新用户失败')
        }
        this.$message.success('更新用户成功')

        // 隐藏添加的对话框
        this.editDilogVisible = false
        // 重新获取列表
        this.gitUserList()
      })
    },
    // 根据id删除用户信息
    async removeUserById(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 用户确定删除 返回字符串confirm
      // 用户取消 返回字符串cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功')
      this.gitUserList()
    },
    // 展示分配角色的对话框
    async setRole(userInfo) {
      this.userInfo = userInfo

      // 获取角色列表
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败!')
      }

      this.rolesList = res.data

      this.setRoleDialogVisible = true
    },
    // 分配角色按钮
    async saveRoleInfo() {
      if (!this.selectedRoleId) {
        return this.$message.error('请选择要分配的角色!')
      }
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`,
        {
          rid: this.selectedRoleId 
        })

      if (res.meta.status !== 200) {
        return this.$message.error('分配的角色失败!')
      }
      this.$message.success('分配的角色成功!')
      this.gitUserList()
      this.setRoleDialogVisible = false
    },
    setRoleDialogClosed() {
      this.selectedRoleId = ''
      this.userInfo = ''
    }
  }
}
</script>

<style lang="less" scoped>

</style>
