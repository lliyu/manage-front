<template>
  <el-row>
    <!--工具条-->
    <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
      <el-form :inline="true" :model="filters">
        <el-form-item>
          <el-input v-model="filters.keyword" placeholder="名称"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button
            type="primary"
            class="el-icon-search"
            :loading="selLoading"
            v-on:click="getLists"
          >查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="success" class="el-icon-plus" @click.native="showForm">新增</el-button>
        </el-form-item>
        <!-- <el-form-item>
					<el-button type="error" class="el-icon-edit" v-on:click="editUser">编辑</el-button>
        </el-form-item>-->
        <el-form-item>
          <el-button type="danger" class="el-icon-delete" @click="deleteData">删除</el-button>
        </el-form-item>
      </el-form>
    </el-col>

    <!--列表-->
    <el-col>
      <el-table
        :data="lists"
        v-loading="listLoading"
        element-loading-text="拼命加载中"
        @selection-change="handleSelectionChange"
        style="width: 100%;"
      >
        <el-table-column type="selection" width="55"></el-table-column>
        <!-- <el-table-column prop="id" label="id"  sortable>
        </el-table-column>-->
        <el-table-column prop="id" label="编码" sortable></el-table-column>
        <el-table-column prop="name" label="名称" sortable></el-table-column>
      </el-table>
    </el-col>
    <el-col>
      <div class="block" style="float: right;margin-right: 10px;margin-top: 10px;">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="startPage"
          :page-sizes="pageSizes"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
        ></el-pagination>
      </div>
    </el-col>
    <el-col :span="2">
      <el-dialog title="新增采集信息" :visible.sync="dialogFormVisible">
        <div style="width:80%;margin: 0 auto">
          <el-form
            :model="info"
            status-icon
            ref="info"
            :inline="false"
            label-width="90px"
            class="demo-ruleForm"
          >
            <el-form-item label="名称" prop="name">
              <el-input type="text" placeholder="采集信息名称" auto-complete="off" v-model="info.name"></el-input>
            </el-form-item>
            <el-form-item label="步骤数" prop="step">
              <el-input type="text" placeholder="采集步骤数" auto-complete="off" v-model="info.step"></el-input>
            </el-form-item>
          </el-form>
        </div>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取 消</el-button>
          <el-button @click="resetForm('info')">重置</el-button>
          <el-button type="primary" @click="addCollect()">确 定</el-button>
        </div>
      </el-dialog>
    </el-col>
  </el-row>
</template>

<script>
import http from "../../utils/http";
import md5 from "js-md5";
import axios from "axios";
let Base64 = require("js-base64").Base64;
export default {
  data() {
    return {
      filters: {
        keyword: ""
      },
      info: {
        name: "",
        step: undefined
      },
      pageSizes: [10, 20, 50],
      startPage: 1,
      pageSize: 10,
      total: 0,
      lists: [],
      total: 0,
      page: 1,
      listLoading: false,
      selLoading: false,
      dialogFormVisible: false,
      selectedList: []
    };
  },
  created() {
    this.getLists();
  },
  methods: {
    // 查询列表数据
    getLists() {
      let _this = this;
      _this.listLoading = true;
      let params = {
        startPage: _this.startPage,
        pageSize: _this.pageSize,
        name: _this.filters.keyword
      };
      http.get("/api/collect/list", params).then(function(response) {
        // console.log(response);
        if (response.status === 200) {
          _this.lists = response.data.list;
          _this.total = response.data.total;
        }
      });
      _this.listLoading = false;
    },
    showForm() {
	//   this.dialogFormVisible = true;   
      this.$router.push({path: '/collect/add?id=' + 1 })
	},
	addCollect(){
		http.get("/api/collect/add", params).then(function(response) {
        if (response.status === 200) {
          this.$router.push({ path: "/collect/add?id=" + response.data });
        }
      });
	},
	
    deleteData() {
      if (this.selectedList.length === 0) {
        this.message(true, "请选择需要删除的菜单", "warning");
        return;
      }
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          http.get("/api/collect/delete", params).then(function(response) {
            // console.log(response);
            if (response.status === 200) {
              this.getLists();
            }
          });
        })
        .catch(() => {
          this.message(true, "已取消删除", "warning");
        });
    },

    // 获取选中集
    handleSelectionChange(val) {
      this.selectedList = [];
      if (val) {
        val.forEach(row => {
          this.selectedList.push(row.id);
        });
      }
    },
    // 每页大小改变时触发
    handleSizeChange(val) {
      this.pageSize = val;
      this.getLists();
    },
    // 当前页码改变时触发
    handleCurrentChange(val) {
      this.startPage = val;
      this.getLists();
    }
  }
};
</script>

<style scoped>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
