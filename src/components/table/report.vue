<template>
  <div class="newFolder" @click="showRightDialog = false">
    <div class="header">
      <el-button
        class="btn"
        type="success"
        size="mini"
        style="background-color:#67c23a;border-color:#67c23a;"
        @click="showCont = true"
      >新建</el-button>
      <el-button
        class="btn"
        type="primary"
        size="mini"
        style="background-color:#3299fe;border-color:#3299fe;"
        @click="showPanel"
      >导入</el-button>
    </div>
    <div class="content">
      <div class="temp-top">
        <div class="left" v-for="(item , index) in myTabs" :key="index">
          <span :class="activeType == 'temp'?'active':''">{{item.label}}</span>
        </div>
      </div>
      <div class="temp-banner">
        <el-row :gutter="20">
          <el-col
            :span="4"
            v-for="(item,index) in myList"
            :key="index"
            @click.native="changeTem(item)"
          >
            <div class="tem" @contextmenu.prevent="showRight($event,item)">
              <img src="/static/imgs/monitor/temp.png" class="img" />
              <p class="title">{{item.value}}</p>
            </div>
          </el-col>
        </el-row>
      </div>
    </div>
    <!-- <div class="dialog" v-show="panelShow">
    <el-dialog title="提示" :visible.sync="panelShow" width="480px" center>
      <span class="dialogT">是否填充系统数据？</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="hidePanel">取 消</el-button>
        <el-button type="primary" @click="onSure" class="subtn">确 定</el-button>
      </span>
    </el-dialog>
    </div>-->

    <div class="dialog" v-show="showCont">
      <el-dialog title="表格模版" :visible.sync="showCont" width="1000px">
        <div class="temp-banner">
          <el-row :gutter="20">
            <el-col
              :span="6"
              v-for="(item,index) in temList"
              :key="index"
              @click.native="changeTem(item)"
            >
              <div class="tem" style="text-align:center">
                <img src="/static/imgs/monitor/temp.png" class="img" />
                <p class="title">{{item.name}}</p>
              </div>
            </el-col>
          </el-row>
        </div>
      </el-dialog>
    </div>
    <div
      class="rightDialog"
      :style="'top:' + top + 'px;left:' + left + 'px'"
      v-if="showRightDialog"
    ></div>
  </div>
</template>

<script>
const { getTemplateThumbnail, deleteSheet } = {};
export default {
  name: "newFolder",
  props: {},
  data() {
    return {
      panelShow: false, // 弹窗
      showCont: false, // 中间缩略图内容
      activeType: "temp",
      myTabs: [{ label: "我的表格", name: "mine" }],
      myList: [],
      tabList: [{ label: "表格模版", name: "temp" }],
      temList: [],
      top: "100",
      left: "200",
      showRightDialog: false,
      param: {},
      menus: [
        {
          name: "删除",
          code: "delete",
          clickEnvent: this.deleteClick
        }
      ]
    };
  },
  mounted() {
    this.getData();
  },
  methods: {
    deleteClick() {
      deleteSheet(this.param).then(res => {
        this.$message({
          message: "删除成功",
          type: "success",
          center: true
        });
        this.getData();
      });
    },
    showRight(e, param) {
      this.param = param;
      this.top = e.clientY;
      this.left = e.clientX;
      this.showRightDialog = true;
    },
    changeTem(val) {
      // 应该增加判断状态的
      this.$router.push({
        path: "/economics/companyReport",
        query: {
          title: val.value ? val.value : val.name ? val.name : "",
          titleId: val.id ? val.id : "",
          belong: val.belong,
          isTemplate: val.isTemplate,
          sheet: val.sheet
        }
      });
    },
    showPanel() {
      // this.panelShow = true
    },
    hidePanel() {
      this.panelShow = false;
    },
    onSure() {
      this.panelShow = false;
    },
    getData() {
      getTemplateThumbnail().then(res => {
        // head为头部信息
        let data = res.data.head || [];
        let list = res.data.context || [];
        this.temList = data;
        this.myList = list;
      });
    }
  }
};
</script>
<style scoped lang="scss">
.newFolder {
  width: 100%;
  padding: 22px;
  background: #fff;
  .header {
    display: flex;
    justify-content: flex-end;
    .btn {
      width: 80px;
      height: 30px;
      line-height: 0px;
    }
  }
  .content {
    border: 1.5px solid #e6e6e6;
    margin: 22px 0;
    padding: 22px 30px;
    .temp-top {
      font-size: 16px;
      color: #666;
      display: flex;
      padding-bottom: 12px;
      justify-content: space-between;
      border-bottom: 1px solid #e6e6e6;
      .left {
        .active {
          color: #4275e7;
        }
      }
      .close {
        width: 14px;
        height: 14px;
        background-repeat: no-repeat !important;
        background-position: center center !important;
        background: url(/static/imgs/monitor/close.png);
      }
    }
    .temp-banner {
      min-height: 450px;
      margin-top: 22px;
      // display: flex;
      .tem {
        color: #666;
        text-align: center;
        .img {
          width: 160px;
          height: 140px;
        }
        .title {
          margin: 4px 0;
        }
      }
    }
  }
  .rightDialog {
    position: absolute;
    width: 120px;
    // background-color: #F00;
  }
}
</style>
<style lang="scss">
.newFolder .dialog {
  .el-dialog__header {
    background-color: rgb(50, 153, 254);
    padding: 12px;
  }
  .el-dialog__title {
    color: rgb(244, 244, 244);
  }
  .el-dialog__headerbtn .el-dialog__close {
    color: rgb(244, 244, 244);
  }
  .el-dialog__headerbtn {
    top: 15px;
  }
  .el-dialog__body {
    padding-bottom: 40px;
  }
}
</style>
