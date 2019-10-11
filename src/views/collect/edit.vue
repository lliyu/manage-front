<template>
  <div class="page">
    <div class="info">
      <div class="name">name: {{ collect.name }}</div>
      <div class="name">step: {{ collect.step }}</div>
    </div>
    <div class="steps">
      <div class="step">
        <el-steps :active="active" finish-status="success">
          <el-step v-for="item in collect.step"></el-step>
        </el-steps>
      </div>
    </div>

    <div class="form">
      <el-form
        :model="info"
        status-icon
        ref="info"
        :inline="false"
        label-width="90px"
        class="demo-ruleForm"
      >
        <el-form-item label="名称" prop="name">
          <el-input type="text" placeholder="该步骤名称" auto-complete="off" v-model="info.name"></el-input>
        </el-form-item>

        <el-form-item label="步骤数" prop="step">
          <el-input type="text" placeholder="采集步骤数" auto-complete="off" v-model="info.index"></el-input>
        </el-form-item>
        <el-form-item label="解析地址" prop="addr">
          <el-input type="text" placeholder="需要解析的地址" auto-complete="off" v-model="info.addr"></el-input>
        </el-form-item>
        <el-form-item
          v-for="(item, index) in itemsForm.items"
          :label="'条目' + index"
          :key="item.key"
          :rules="{
      required: true, message: '域名不能为空', trigger: 'blur'
    }"
        >
          <el-input v-model="item.name" placeholder="名称"></el-input>
          <el-input v-model="item.value" placeholder="解析表达式"></el-input>
          <el-button @click.prevent="removeItem(item)">删除</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="btn">
      <el-button @click="addItem">新增条目</el-button>
      <el-button type="primary" @click="test">测试</el-button>
      <el-button type="primary" :disabled="predisabled" @click="pre">上一步</el-button>
      <el-button type="primary" :disabled="nextdisabled" @click="next">下一步</el-button>
      <el-button type="primary" :disabled="commitBtn" @click="addItemStep" >完成</el-button>
    </div>
    <el-dialog
  title="提示"
  :visible.sync="dialogVisible"
  width="30%"
  >
  <span>
      <pre>
      {{ message }}
      </pre>
      </span>
  <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>

<script>
import http from "../../utils/http";
import { MessageBox } from 'element-ui';
export default {
  data() {
    return {
      active: 0,
      index: 2,
      id: undefined,
      dialogVisible: false,
      message: undefined,
      collect: {
        name: undefined,
        step: undefined,
        index: undefined
      },
      steps: [],
      info: {
        name: "baidu",
        addr: "http://www.baidu.com/s?ie=UTF-8&wd=baidu",
        value: undefined,
        index: 1,
        collectId: undefined
      },
      itemsForm: {
        items: [
          {
            value: "//h3[@class='t']/a/allText()",
            name: "item"
          }
        ]
      },
      predisabled: true,
      nextdisabled: false,
      commitBtn: true
    };
  },
  created() {
    var href = window.location.href;
    if (href.indexOf("?") != -1) {
      var str = href.split("?")[1];
      this.id = str.split("=")[1];
    }
    this.collectInfo();
  },
  watch:{
      active(val){
          if(this.collect.step-1 === this.active){
              this.commitBtn = false;
          }else{
              this.commitBtn = true;
          }
      }
  },
  methods: {
    pre() {
        this.active--;
      if (this.active == 0) {
        this.predisabled = true;
        this.nextdisabled = false;
      }else{
          this.predisabled = false;
          this.nextdisabled = false;
      }
      
      var info = this.steps[this.active];
      //todo
    },
    next() {
        this.active++;
      if (this.active == this.collect.step - 1) {
          this.predisabled = false;
        this.nextdisabled = true;
      }else{
          this.predisabled = false;
          this.nextdisabled = false;
      }
      
      //将当前步骤存储
      this.info.value = JSON.stringify(this.itemsForm.items);
      this.info.collectId = this.id;
      this.steps.push(JSON.parse(JSON.stringify(this.info)));
      this.info.name = undefined;
      this.info.addr = undefined;
      this.info.index = undefined;
      this.info.value = undefined;
    //   this.itemsForm.items = {
    //         value: "//h3[@class='t']/a/allText()",
    //         name: "item"
    //       };
      console.log(this.steps);
    },
    collectInfo() {
      let _this = this;
      var params = {
        id: _this.id
      };
      http.get("/api/collect/info", params).then(function(res) {
        if (res.status == 200) {
          _this.collect = res.data.data;
        }
      });
    },
    addItemStep(){
        let _this = this;
        //将当前步骤存储
      _this.info.value = JSON.stringify(_this.itemsForm.items);
      _this.info.collectId = _this.id;
      _this.steps.push(JSON.parse(JSON.stringify(_this.info)));
        let param = {
            steps: JSON.stringify(_this.steps)
        }
        http.post("/api/step/add", param).then(function(res){
            if(res.status == 200){
                _this.$notify({
                title: '提示',
                message: '添加成功'
                });
                //返回列表
            }
        });

    },
    removeItem(item) {
      var index = this.itemsForm.items.indexOf(item);
      if (index !== -1) {
        this.itemsForm.items.splice(index, 1);
      }
    },
    addItem() {
      this.itemsForm.items.push({
        value: undefined,
        name: undefined
      });
      console.log(this.itemsForm.items);
    },
    test() {
        let _this = this;
      _this.info.value = JSON.stringify(_this.itemsForm.items);
      http.post("/api/collect/test", _this.info).then(function(res) {
        console.log(res);
        // _this.message = JSON.stringify(res.data.data);
        _this.message = res.data.data;
        _this.dialogVisible = true;
      });
    }
  }
};
</script>

<style scoped>
.page {
  width: 70%;
  padding-top: 20px;
}
.steps {
  width: 600px;
  height: 50px;
}
/* .step {
  margin-top: 20px;
  float: left;
} */
/* .next {
  float: left;
} */
.form {
  margin-top: 30px;
}
</style>