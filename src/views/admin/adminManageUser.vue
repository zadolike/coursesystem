<template>
  <div>
    <el-form :inline="true">
      <el-form-item>
        <el-input
            v-model="searchForm.sno"
            placeholder="用户名"
            clearable
        >
        </el-input>
      </el-form-item>

      <el-form-item>
        <el-button @click="getUserList">搜索</el-button>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" @click="dialogVisible = true">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-popconfirm title="确定批量删除吗？" @confirm="delHandle(null)">
          <el-button type="danger" slot="reference" :disabled="delBtlStatu">批量删除</el-button>
        </el-popconfirm>
      </el-form-item>
    </el-form>

    <el-table
        ref="multipleTable"
        :data="tableData"
        tooltip-effect="dark"
        style="width: 100%"
        border
        stripe
        @selection-change="handleSelectionChange">

      <el-table-column
          type="selection"
          width="55">
      </el-table-column>


      <el-table-column
          prop="id"
          label="ID"
          width="120">
      </el-table-column>

      <el-table-column
          prop="sname"
          label="姓名">
      </el-table-column>
      <el-table-column
          prop="sno"
          label="学号">
      </el-table-column>
      <el-table-column
          prop="sex"
          label="性别">
      </el-table-column>
      <el-table-column
          prop="pwd"
          label="密码">
      </el-table-column>


      <el-table-column
          prop="icon"
          width="260px"
          label="操作">

        <template slot-scope="scope">
          <el-button type="text" @click="editHandle(scope.row.id)">编辑</el-button>
          <el-divider direction="vertical"></el-divider>

          <template>
            <el-popconfirm title="确定删除吗？" @confirm="delHandle(scope.row.id)">
              <el-button type="text" slot="reference">删除</el-button>
            </el-popconfirm>
          </template>

        </template>
      </el-table-column>

    </el-table>

    <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        layout="total, sizes, prev, pager, next, jumper"
        :page-sizes="[10, 20, 50, 100]"
        :current-page="current"
        :page-size="size"
        :total="total">
    </el-pagination>


    <!--新增对话框-->
    <el-dialog
        title="提示"
        :visible.sync="dialogVisible"
        width="600px"
        :before-close="handleClose">

      <el-form :model="editForm" :rules="editFormRules" ref="editForm">
        <el-form-item label="ID" prop="id" label-width="100px">
          <el-input v-model="editForm.id" autocomplete="off" disabled></el-input>
        </el-form-item>
          <el-form-item label="学号" prop="sno" label-width="100px">
            <el-input v-model="editForm.sno" autocomplete="off" ></el-input>
          </el-form-item>
        <el-form-item label="姓名" prop="sname" label-width="100px">
          <el-input v-model="editForm.sname" autocomplete="off"></el-input>
<!--          <el-alert-->
<!--              title="初始密码为888888"-->
<!--              :closable="false"-->
<!--              type="info"-->
<!--              style="line-height: 12px;"-->
<!--          ></el-alert>-->
        </el-form-item>

        <el-form-item label="性别"  prop="sex" label-width="100px">
          <el-radio-group v-model="editForm.sex">
            <el-radio :label="0">男</el-radio>
            <el-radio :label="1">女</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="密码"  prop="pwd" label-width="100px">
          <el-input v-model="editForm.pwd" autocomplete="off"></el-input>
        </el-form-item>


      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="resetForm('editForm')">取 消</el-button>
        <el-button type="primary" @click="submitForm('editForm')">确 定</el-button>
      </div>
    </el-dialog>


  </div>
</template>

<script>
export default {
  name: "adminManageUser",
  data() {
    return {
      searchForm: {},
      delBtlStatu: true,
      total: 0,
      size: 10,
      current: 1,
      dialogVisible: false,
      editForm: {
        id:"",
        sno:"",
        sname:"",
        sex:0,
        pwd:""
      },
      tableData: [],
      editFormRules: {
        sname: [
          {required: true, message: '请输入姓名', trigger: 'blur'}
        ],
        sex: [
          {required: true, message: '请输入性别', trigger: 'blur'}
        ],
        pwd: [
          {required: true, message: '请输入密码', trigger: 'blur'}
        ]
      },
      multipleSelection: [],
      roleDialogFormVisible: false,
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      roleForm: {},
      roleTreeData:  [],
      treeCheckedKeys: [],
      checkStrictly: true
    }
  },
  created() {
    this.getUserList()
    this.$axios.get("/student/list").then(res => {
      this.roleTreeData = res.data.data.records
    })
  },
  methods: {
    toggleSelection(rows) {
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row);
        });
      } else {
        this.$refs.multipleTable.clearSelection();
      }
    },
    handleSelectionChange(val) {
      console.log("勾选")
      console.log(val)
      this.multipleSelection = val;
      this.delBtlStatu = val.length == 0
    },
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.size = val
      this.getUserList()
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.current = val
      this.getUserList()
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
      this.dialogVisible = false
      this.editForm = {}
    },
    handleClose() {
      this.resetForm('editForm')
    },
    getUserList() {
      this.$axios.get("/student/list/", {
        params: {
          sno: this.searchForm.sno,
          current: this.current,
          size: this.size
        }
      }).then(res => {
        this.tableData = res.data.data.records
        this.size = res.data.data.size
        this.current = res.data.data.current
        this.total = res.data.data.total
      })
    },
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          console.log(this.editForm)
          this.$axios.post('/student' + (this.editForm.id?'/update' : '/save'), this.editForm)
              .then(res => {
                this.$message({
                  showClose: true,
                  message: '恭喜你，操作成功',
                  type: 'success',
                  onClose:() => {
                    this.getUserList()
                  }
                });
                this.dialogVisible = false
              })
        } else {
          console.log('error submit!!');
          return false;
        }
      });
    },
    editHandle(id) {
      console.log(id)
      this.$axios.get('/student/Info/' + id).then(res => {
        this.editForm = res.data.data
        if (res.data.data.sex=="男"){
            this.editForm.sex = 0
        }
        else{
          this.editForm.sex = 1
        }
        this.dialogVisible = true
      })
    },
    delHandle(id) {
      var ids = []
      if (id) {
        ids.push(id)
      } else {
        this.multipleSelection.forEach(row => {
          ids.push(row.id)
        })
      }
      console.log(ids)
      this.$axios.post("/student/delete", ids).then(res => {
        this.$message({
          showClose: true,
          message: '恭喜你，操作成功',
          type: 'success',
          onClose:() => {
            this.getUserList()
          }
        });
      })
    },
  }
}
</script>

<style scoped>
.el-pagination {
  float: right;
  margin-top: 22px;
}
</style>
