<template>
  <div style="background-color: rgb(240, 240, 240)">
    <el-drawer
      title="请输入新增VIP信息"
      v-model="dialog"
      direction="ltr"
      custom-class="demo-drawer"
      ref="drawer"
      @close="resetForm('form')"
    >
      <div class="demo-drawer__content">
        <el-form :model="form" ref="form">
          <el-form-item
            label="用户名"
            :label-width="formLabelWidth"
            prop="username"
            :rules="[
              { required: true, message: '用户名不能为空', trigger: 'blur' },
            ]"
          >
            <el-input
              v-model="form.username"
              autocomplete="off"
              placeholder="请输入用户名"
            ></el-input>
          </el-form-item>
          <el-form-item
            label="邮箱"
            :label-width="formLabelWidth"
            prop="email"
            :rules="[
              { required: true, message: '邮箱不能为空', trigger: 'blur' },
            ]"
          >
            <el-input
              v-model="form.email"
              autocomplete="off"
              placeholder="请输入邮箱"
            ></el-input>
          </el-form-item>

          <el-form-item
            label="VIP等级"
            :label-width="formLabelWidth"
            prop="vip"
            :rules="[
              { required: true, message: '等级不能为空', trigger: 'change' },
            ]"
          >
            <el-select v-model="form.vip" placeholder="请选择等级">
              <el-option label="1" value="1"></el-option>
              <el-option label="2" value="2"></el-option>
              <el-option label="3" value="3"></el-option>
              <el-option label="4" value="4"></el-option>
            </el-select>
          </el-form-item>


        </el-form>
        <div class="demo-drawer__footer" style="margin-left: 80px">
          <el-button @click="cancelForm">取 消</el-button>
          <el-button type="primary" @click="submitForm('form')"
            >确 定</el-button
          >
        </div>
      </div>
    </el-drawer>

    <div>
      <el-breadcrumb separator-class="el-icon-d-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }"
          >回到首页
        </el-breadcrumb-item>
        <el-breadcrumb-item :to="{ path: '/admin1' }"
          >用户管理
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div style="padding: 15px">
      <el-table
        :data="
          tableData.filter(
            (data) =>
              !search ||
              data.username.toLowerCase().includes(search.toLowerCase())
          )
        "
        style="width: 100%"
        stripe
      >
        <el-table-column label="用户名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="会员等级" prop="vip"></el-table-column>
        <el-table-column align="right">
          <template #header>
            <el-input
              v-model="search"
              size="mini"
              placeholder="输入关键字搜索"
              clearable
            />
          </template>
          <template #default="scope">
            <el-button size="mini" @click="handleEdit(scope.row)"
              >编辑
            </el-button>
            <el-popconfirm
              title="确定删除吗？"
              @confirm="handleDelete(scope.row.id)"
            >
              <template #reference>
                <el-button size="mini" type="danger">删除 </el-button>
              </template>
            </el-popconfirm>
          </template>
        </el-table-column>
      </el-table>
      <div style="margin: 10px 0">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-sizes="[5, 10, 20, 50, 250]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
        >
        </el-pagination>
      </div>
      <div style="margin: 10px 0">
        <el-button
          type="primary"
          @click="
            dialog = true;
            this.form = {};
            this.flag = false;
          "
          >新增</el-button
        >
        <el-button type="primary">导入</el-button>
        <el-button type="primary">导出</el-button>
      </div>
    </div>
  </div>
</template>

<script>
import request from "../utils/request";

export default {
  name: "AdminVIP",
  components: {},
  data() {
    return {
      currentPage: 1,
      total: 0,
      tableData: [],
      search: "",
      pageSize: 10,
      form: {
        username: "",
        email: "",
        vip: ""
      },
      flag: true,
      dialog: false,
      formLabelWidth: "80px",
    };
  },
  created() {
    this.load();
  },
  methods: {
    load() {
      request
        .get("http://localhost:9090/uservip", {
          params: {
            pageNum: this.currentPage,
            pageSize: this.pageSize,
            search: this.search,
          },
        })
        .then((res) => {
          console.log(res);
          this.tableData = res.data.records;
          this.total = res.data.total;
        });
    },
    handleEdit(row) {
      this.form = JSON.parse(JSON.stringify(row));
      this.flag = true;
      this.dialog = true;
      console.log(row);
      console.log(this.form);
    },
    handleDelete(id) {
      request.delete("http://localhost:9090/uservip/" + id).then((res) => {
        if (res.code === "0") {
          this.$message({
            type: "success",
            message: "删除成功",
          });
        } else {
          this.$message({
            type: "error",
            message: res.msg,
          });
        }
        this.load();
      });
    },
    handleSizeChange(pageSize) {
      this.pageSize = pageSize;
      this.load();
    },
    handleCurrentChange(pageNum) {
      this.currentPage = pageNum;
      this.load();
    },

    cancelForm() {
      this.dialog = false;
      this.resetFields();
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          if (this.flag) {
            request.put("http://localhost:9090/uservip", this.form).then((res) => {
              console.log(res);
              if (res.code === "0") {
                this.$message({
                  type: "success",
                  message: "编辑成功",
                });
              } else {
                this.$message({
                  type: "error",
                  message: res.msg,
                });
              }
              this.load(); //刷新表格数据
              this.dialog = false; //关闭弹窗
            });
          } else {
            request
              .post("http://localhost:9090/uservip", this.form)
              .then((res) => {
                console.log(res);
                if (res.code === "0") {
                  this.$message({
                    type: "success",
                    message: "新增成功",
                  });
                } else {
                  this.$message({
                    type: "error",
                    message: res.msg,
                  });
                }
                this.load(); //刷新表格数据
                this.dialog = false; //关闭弹窗
              });
          }
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
  },
};
</script>

<style>
</style>