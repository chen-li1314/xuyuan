<template>
  <div class="table">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item><i class="el-icon-lx-friend"></i> {{title}}</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <div class="handle-box">
        <el-input v-model="selectUsername" placeholder="搜索用户名" class="handle-input mr10"></el-input>
        <el-button type="primary" icon="search" @click="search">搜索</el-button>
        <el-button type="primary" icon="search" @click="handleAdd">添加</el-button>
      </div>
      <el-table :data="tableData" border class="table" ref="multipleTable" @selection-change="handleSelectionChange">
        <el-table-column prop="username" label="用户名" sortable>
        </el-table-column>
        <el-table-column prop="name" label="姓名">
        </el-table-column>
        <el-table-column prop="role" label="角色" :formatter="calcRole">
        </el-table-column>
        <el-table-column prop="createdAt" label="创建时间">
        </el-table-column>
        <el-table-column label="操作" width="180" align="center">
          <template slot-scope="scope">
            <el-button type="text" icon="el-icon-edit" @click="handleEdit(scope.$index,scope.row)">修改</el-button>
            <el-button type="text" icon="el-icon-delete" class="red" @click="handleDelete(scope.$index,scope.row)">删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination background @current-change="handleCurrentChange" :page-size="rows" layout="prev, pager, next"
                       :total="count">
        </el-pagination>
      </div>
    </div>

    <!-- 添加弹出框 -->
    <el-dialog title="添加管理员" :visible.sync="addVisible" width="30%">
      <el-form ref="form" :model="addForm" label-width="60px">
        <el-form-item label="用户名">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="姓名">
          <el-input v-model="addForm.name"></el-input>
        </el-form-item>
        <el-form-item label="角色">
          <el-select v-model="addForm.role" placeholder="请选择" style="width: 100%;">
            <el-option label="超级管理员" :value="1"></el-option>
            <el-option label="普通管理员" :value="2"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
                <el-button @click="addVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveAdd">确 定</el-button>
            </span>
    </el-dialog>

    <!-- 编辑弹出框 -->
    <el-dialog title="修改管理员" :visible.sync="editVisible" width="30%">
      <el-form ref="form" :model="editForm" label-width="60px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码">
          <el-input v-model="editForm.password" show-password></el-input>
        </el-form-item>
        <el-form-item label="姓名">
          <el-input v-model="editForm.name"></el-input>
        </el-form-item>
        <el-form-item label="角色">
          <el-select v-model="editForm.role" placeholder="请选择" style="width: 100%;">
            <el-option label="超级管理员" :value="1"></el-option>
            <el-option label="普通管理员" :value="2"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
    </el-dialog>

    <!-- 删除提示框 -->
    <el-dialog title="删除管理员提示" :visible.sync="delVisible" width="300px" center>
      <div class="del-dialog-cnt">删除不可恢复，是否确定删除？</div>
      <span slot="footer" class="dialog-footer">
                <el-button @click="delVisible = false">取 消</el-button>
                <el-button type="primary" @click="deleteRow">确 定</el-button>
            </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        title: this.$route.meta.title,
        tableData: [],
        cur_page: 1,
        rows: 2,
        multipleSelection: [],
        select_cate: '',
        select_word: '',
        del_list: [],
        addVisible: false,
        editVisible: false,
        delVisible: false,
        editForm: {
          username: '',
          password: '',
          name: '',
          role: null
        },
        addForm: {
          username: '',
          password: '',
          name: '',
          role: 1
        },
        idx: - 1,
        count: 0,
        selectUsername: ''
      }
    },
    created () {
      this.getData ();
    },
    methods: {
      calcRole (row, column) {
        return row[column.property] == 1 ? '超级管理员' : '普通管理员'
      },
      // 分页导航
      handleCurrentChange (val) {
        this.cur_page = val;
        this.getData ();
      },
      // 获取列表数据
      getData (search) {
        this.fetchData ({
          method: 'get',
          url: '/api/admin',
          params: {
            page: search ? 1 : this.cur_page,
            rows: this.rows,
            username: this.selectUsername
          },
          success: data => {
            this.tableData = data.list;
            this.count = data.count;
          }
        })
      },
      search () {
        this.getData(true);
      },
      handleAdd () {
        this.addVisible = true;
      },
      handleEdit (index, row) {
        this.idx = index;
        // const item = this.tableData[index];
        // console.log(index);
        this.fetchData ({
          method: 'get',
          url: '/api/admin/' + row.id,
          success: data => {
            // console.log("编辑data",typeof(data.role));
            this.editForm = {
              name: data.name,
              username: data.username,
              password: data.password,
              role: data.role
            };
            console.log("编辑data",this.editForm.role);
          }
        });
        this.editVisible = true;
      },

      handleDelete (index, row) {
        this.idx = index;
        this.delVisible = true;
      },
      delAll () {
        const length = this.multipleSelection.length;
        let str = '';
        this.del_list = this.del_list.concat (this.multipleSelection);
        for (let i = 0; i < length; i ++) {
          str += this.multipleSelection[i].name + ' ';
        }
        this.$message.error ('删除了' + str);
        this.multipleSelection = [];
      },
      handleSelectionChange (val) {
        this.multipleSelection = val;
      },
      // 添加
      saveAdd () {
        this.fetchData ({
          method: 'post',
          url: '/api/admin',
          data: {
            username: this.addForm.username,
            password: this.addForm.password,
            name: this.addForm.name,
            role: this.addForm.role
          },
          success: data => {
            this.$message.success (`添加成功`);
            this.addVisible = false;
            this.getData ();
          }
        });

      },
      // 修改
      saveEdit () {
        this.fetchData ({
          method: 'put',
          url: '/api/admin',
          data: {
            id: this.tableData[this.idx].id,
            username: this.editForm.username,
            password: this.editForm.password,
            name: this.editForm.name,
            role: this.editForm.role
          },
          success: data => {
            this.$message.success (`修改成功`);
            this.editVisible = false;
            this.getData ();
          }
        });

        // this.$set(this.tableData, this.idx, this.form);
        // this.editVisible = false;

      },
      // 确定删除
      deleteRow () {
        this.fetchData ({
          method: 'delete',
          url: '/api/admin',
          data: {
            id: this.tableData[this.idx].id
          },
          success: data => {
            this.$message.success ('删除成功');
            this.delVisible = false;
            this.getData ();
          }
        });
      }
    }
  }

</script>

<style scoped>
  .handle-box {
    margin-bottom: 20px;
  }

  .handle-select {
    width: 120px;
  }

  .handle-input {
    width: 300px;
    display: inline-block;
  }

  .del-dialog-cnt {
    font-size: 16px;
    text-align: center
  }

  .table {
    width: 100%;
    font-size: 14px;
  }

  .red {
    color: #ff0000;
  }

  .mr10 {
    margin-right: 10px;
  }
</style>
