<template>
  <div class="report">
    <div class="jingji">
      <p class="titleN">
        <span>{{ project.title || '--' }}</span>
        <el-button
          type="primary"
          size="mini"
          @click="download"
          style="float: right;margin-right: 63px;"
        >
          <img style="width: 12px;padding-right: 6px" src="/static/imgs/binjiang/export.png" />报表下载
        </el-button>
        <el-button
          type="success"
          size="mini"
          @click="exportData"
          style="float: right;margin-right: 8px;background-color:#67c23a;border-color:#67c23a;"
        >
          <img style="width: 12px;padding-right: 6px" src="/static/imgs/binjiang/import.png" />自动导入
        </el-button>
        <el-button type="primary" size="mini" @click="onSave" style="float: right;">
          <img style="width: 12px;padding-right: 6px" src="/static/imgs/binjiang/save.png" />保存报表
        </el-button>
      </p>
      <el-table :style="'overflow-y: auto;height:'+ maxHeight" :data="tableData">
        <el-table-column
          v-for="item in tableColumn"
          :prop="item.prop"
          :key="item.prop"
          :width="item.width"
          :label="item.label"
        ></el-table-column>
        <el-table-column
          prop="operation"
          label="操作"
          width="120px"
          v-if="tableColumn && tableColumn.length > 0"
        >
          <template slot-scope="scope" v-if="scope.row.indicator_value">
            <el-button plain size="mini" @click="modify(scope.row)">修改</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="modify">
      <el-dialog title="修改" :visible.sync="dialogVisible" width="40%" center>
        <div class="dialogT">
          <p>指标名称：{{ inTitle }}</p>
          <div>
            指标值：
            <el-input style="width: 50%;" v-model="inValue" placeholder="请输入内容"></el-input>
          </div>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" class="subtn" @click="saveContent()">确 定</el-button>
        </span>
      </el-dialog>
    </div>
    <div class="dialog" v-show="panelShow">
      <el-dialog title="提示" :visible.sync="panelShow" width="40%" center>
        <span class="dialogT">是否填充系统数据？</span>
        <span slot="footer" class="dialog-footer">
          <el-button @click="hidePanel">取 消</el-button>
          <el-button type="primary" @click="onSure" class="subtn">确 定</el-button>
        </span>
      </el-dialog>
    </div>
    <div class="saveDialog">
      <el-dialog title="保存" :visible.sync="saveDialogVisible" width="40%" center>
        <div class="dialogT">
          <!-- <p>请输入文件名称</p> -->
          <div>
            <el-input style="width: 50%;" v-model="saveTitle" placeholder="请输入文件名称"></el-input>
          </div>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="saveDialogVisible = false">取 消</el-button>
          <el-button type="primary" class="subtn" @click="onSaveData()">确 定</el-button>
        </span>
      </el-dialog>
    </div>
  </div>
</template>
<script>
const { getAllContext, createSheet, exportSheet } = {};
const project = {
  title: "指挥部 ",
  titleId: ""
  // type: '季报'
};
export default {
  name: "report",
  components: {},
  data() {
    const meta = this.$route.meta || {};
    return {
      saveDialogVisible: false,
      saveTitle: "",
      height: meta.height || 1024,
      maxHeight: "",
      belong: 1,
      isTemplate: 0,
      isTemplateData: 0,
      panelShow: false, // 自动导入弹窗
      dialogVisible: false,
      inTitle: "",
      inValue: "",
      activeName: "month",
      row: "",
      labels: [
        {
          label: "报表导出",
          name: "month"
        }
      ],
      tableColumn: [{}],
      tableData: [],
      partTableData: {},
      project
    };
  },
  created() {
    // this.resize()
    // this.getParams()
    this.getData();
  },
  mounted() {
    let that = this;
    window.onresize = () => {
      if (!that.timer) {
        that.timer = true;
        setTimeout(() => {
          that.resize();
        }, 100);
      }
    };
  },
  methods: {
    // 最大高度
    resize() {
      const height = document.documentElement.clientHeight;
      this.maxHeight = height - 258 + "px";
      this.timer = false;
    },
    modify(row) {
      this.partTableData[row.orderNo] = row;
      this.inTitle = row.indicator_name;
      this.inValue = row.unit == "--" ? "" : row.unit;
      this.row = row;
      this.dialogVisible = true;
    },
    exportData() {
      this.panelShow = true;
    },
    download(url, fileName) {
      let param = {
        belong: this.belong,
        isTemplate: this.isTemplate,
        sheet: this.sheet
      };
      exportSheet(param).then(res => {
        let url = res.data;
        window.open(url);
      });
    },
    saveContent() {
      this.dialogVisible = false;
      this.inTitle = this.row.indicator_name;
      this.row.unit = this.inValue;
    },
    getData() {
      let params = {
        isTemplate: this.isTemplate,
        belong: this.belong,
        sheet: this.sheet
      };
      getAllContext(params).then(res => {
        let widthArr = ["480px", "", "", ""];
        let data = res.data || [];
        let tabColumn = data.head || [];
        let tableData = data.context || [];
        this.tableColumn = tabColumn.map((item, index) => {
          return {
            label: item.name,
            prop: item.code,
            belong: item.belong,
            width: widthArr[index]
          };
        });
        if (this.isTemplateData == 1) {
          tableData.map(item => {
            item.unit = item.indicator_value ? "--" : "";
          });
        }
        this.tableData = tableData;
      });
    },
    getParams() {
      this.project.title = this.$route.query.title || "";
      this.project.titleId = this.$route.query.titleId || "";
      this.belong = this.$route.query.belong || "";
      this.isTemplate = this.$route.query.isTemplate || "";
      this.isTemplateData = this.isTemplate || 0;
      this.sheet = this.$route.query.sheet || "";
    },
    hidePanel() {
      this.panelShow = false;
    },
    onSure() {
      this.panelShow = false;
      this.isTemplateData = 0;
      this.getData();
    },
    // 保存按钮
    onSave() {
      if (this.isTemplate == 0) {
        this.saveTitle = this.project.title;
      }
      this.saveDialogVisible = true;
    },
    onSaveData() {
      this.saveDialogVisible = false;
      let data = this.tableData;
      if (this.isTemplate == 0) {
        data = [];
        if (this.partTableData) {
          for (let i in this.partTableData) {
            data.push(this.partTableData[i]);
          }
        }
      }
      let params = {
        belong: this.belong,
        isTemplate: this.isTemplate,
        result: data,
        sheet: this.sheet,
        title: this.saveTitle,
        titleId: this.project.titleId
      };
      createSheet(JSON.stringify(params)).then(res => {
        if (res.errorCode == 0) {
          this.$message({
            message: "保存成功！",
            type: "success",
            center: true
          });
        } else {
          this.$message({
            message: "保存失败！",
            type: "error",
            center: true
          });
        }
      });
    }
  },
  watch: {
    $route: "getParams"
  }
};
</script>
<style lang="scss" scoped>
.report {
  .goBack {
    color: rgb(153, 153, 153);
    cursor: pointer;
    margin-bottom: 18px;
  }

  .jingji {
    margin: 0 auto;
    background: #fff;
    padding: 15px 30px 15px 30px;

    p.titleN {
      color: rgb(51, 51, 51);
      font-weight: bold;
      font-size: 16px;
    }
  }

  .dialogT {
    width: 60%;
    margin: 0 auto;
    color: rgb(102, 102, 102);
    margin-top: 30px;
  }
}
</style>
<style>
.jingji .el-table th,
.jingji .el-table td {
  padding-left: 20px;
  color: rgb(102, 102, 102);
}

.jingji .el-table {
  border-right: 1px solid #ebeef5;
  border-left: 1px solid #ebeef5;
}

.report .el-tabs__nav-wrap::after {
  height: 0px;
}

.report .el-tabs__item:hover,
.report .el-tabs__item.is-active {
  color: #3299fe;
}

.report .el-tabs__active-bar {
  background-color: #3299fe;
}

.report .el-tabs__item {
  color: rgb(102, 102, 102);
  font-size: 18px;
}

.jingji .el-button {
  color: #3299fe;
  border: 1px solid #3299fe;
}

.jingji .el-button.is-plain:hover,
.jingji .el-button.is-plain:focus {
  color: #3299fe;
  border: 1px solid #3299fe;
}

.jingji .titleN .el-button {
  color: #fff;
  background-color: #3299fe;
}

/* .jingji .el-table th.is-leaf,
  .jingji .el-table td {
    border: 1px solid #ebeef5;
  } */
.report .el-dialog__header {
  background-color: #3299fe;
  padding: 12px;
}

.report .el-dialog__title {
  color: rgb(244, 244, 244);
}

.report .el-dialog__headerbtn .el-dialog__close {
  color: rgb(244, 244, 244);
}

.report .el-dialog__headerbtn {
  top: 15px;
}

.report .el-dialog__body {
  padding-bottom: 40px;
}

.subtn {
  background-color: #3299fe;
  border-color: #3299fe;
}
</style>
