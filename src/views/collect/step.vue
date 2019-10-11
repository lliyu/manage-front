<template>
  <div class="card">
    <el-row>
  <el-col :span="8" v-for="(o, index) in 6" :key="o" :offset="index > 0 ? 2 : 0">
    <el-card :body-style="{ padding: '0px' }">
      <img src="https://i1.hdslb.com/bfs/live/e8631214a846035037b059d2d1c8f603312b4dcc.jpg@1e_1c_100q.webp" class="image">
      <div style="padding: 14px;">
        <span>好吃的汉堡</span>
        <div class="bottom clearfix">
          <time class="time">2017-09-09</time>
          <el-button type="text" class="button">操作按钮</el-button>
        </div>
      </div>
    </el-card>
  </el-col>
</el-row>
  </div>
</template>

<style>
  /* .text {
    font-size: 14px;
  }

  .item {
    margin-bottom: 18px;
  }

  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }
  .clearfix:after {
    clear: both
  }

  .box-card {
    width: 480px;
  } */
</style>