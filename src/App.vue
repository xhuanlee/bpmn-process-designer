<template>
  <div id="app">
    <my-process-designer
      :key="`designer-${reloadIndex}`"
      v-model="xmlString"
      v-bind="controlForm"
      keyboard
      ref="processDesigner"
      prefix="flowable"
      @element-click="elementClick"
      @init-finished="initModeler"
    />
    <my-properties-panel
      :key="`penal-${reloadIndex}`"
      :bpmn-modeler="modeler"
      :prefix="controlForm.prefix"
      class="process-panel"
    />

    <!-- demo config -->
    <div class="demo-control-bar">
      <div class="open-control-dialog" @click="controlDrawerVisible = true">
        <i class="el-icon-setting"></i>
      </div>
    </div>
    <el-drawer
      :visible.sync="controlDrawerVisible"
      size="400px"
      title="偏好设置"
      append-to-body
      destroy-on-close
    >
      <el-form
        :model="controlForm"
        size="small"
        label-width="100px"
        class="control-form"
        @submit.native.prevent
      >
        <el-form-item label="流程ID">
          <el-input
            v-model="controlForm.processId"
            @change="reloadProcessDesigner"
          />
        </el-form-item>
        <el-form-item label="流程名称">
          <el-input
            v-model="controlForm.processName"
            @change="reloadProcessDesigner"
          />
        </el-form-item>
        <el-form-item label="流转模拟">
          <el-switch
            v-model="controlForm.simulation"
            inactive-text="停用"
            active-text="启用"
            @change="reloadProcessDesigner"
          />
        </el-form-item>
        <el-form-item label="禁用双击">
          <el-switch
            v-model="controlForm.labelEditing"
            inactive-text="停用"
            active-text="启用"
            @change="changeLabelEditingStatus"
          />
        </el-form-item>
        <el-form-item label="隐藏label">
          <el-switch
            v-model="controlForm.labelVisible"
            inactive-text="停用"
            active-text="启用"
            @change="changeLabelVisibleStatus"
          />
        </el-form-item>
        <el-form-item label="流程引擎">
          <el-radio-group
            v-model="controlForm.prefix"
            @change="reloadProcessDesigner"
          >
            <el-radio label="camunda">camunda</el-radio>
            <el-radio label="flowable">flowable</el-radio>
            <el-radio label="activiti">activiti</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="工具栏">
          <el-radio-group v-model="controlForm.headerButtonSize">
            <el-radio label="mini">mini</el-radio>
            <el-radio label="small">small</el-radio>
            <el-radio label="medium">medium</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <br />
      <p style="color: #999999">
        注：activiti 好像不支持表单配置，控制台可能会报错
      </p>
      <p style="color: #999999">
        更多配置请查看源码：<a
          href="https://github.com/miyuesc/bpmn-process-designer"
          >MiyueSC/bpmn-process-designer</a
        >
      </p>
    </el-drawer>
  </div>
</template>

<script>
import translations from "@/translations";
// 自定义渲染（隐藏了 label 标签）
import CustomRenderer from "@/modules/custom-renderer";
// 自定义元素选中时的弹出菜单（修改 默认任务 为 用户任务）
import CustomContentPadProvider from "../package/process-designer/plugins/content-pad";
// 自定义左侧菜单（修改 默认任务 为 用户任务）
import CustomPaletteProvider from "../package/process-designer/plugins/palette";
import xmlObj2json from "./utils/xml2json";
// 自定义侧边栏
// import MyProcessPanel from "../package/process-panel/ProcessPanel";

export default {
  name: "App",
  components: {},
  data() {
    return {
      xmlString: "",
      modeler: null,
      reloadIndex: 0,
      controlDrawerVisible: false,
      translationsSelf: translations,
      controlForm: {
        processId: "",
        processName: "",
        simulation: true,
        labelEditing: false,
        labelVisible: false,
        prefix: "activiti",
        headerButtonSize: "mini",
        // additionalModel: []
        additionalModel: [CustomContentPadProvider, CustomPaletteProvider],
      },
      addis: {},
    };
  },
  created() {
    // console.log(this.translationsSelf);
    this.requestUserInfo();
  },
  methods: {
    initModeler(modeler) {
      setTimeout(() => {
        this.modeler = modeler;
        console.log(modeler);
      }, 10);
    },
    reloadProcessDesigner() {
      this.controlForm.additionalModel = [];
      for (let key in this.addis) {
        if (this.addis[key]) {
          this.controlForm.additionalModel.push(this.addis[key]);
        }
      }
      this.reloadIndex += 1;
      this.modeler = null; // 避免 panel 异常
    },
    changeLabelEditingStatus(status) {
      this.addis.labelEditing = status
        ? { labelEditingProvider: ["value", ""] }
        : false;
      this.reloadProcessDesigner();
    },
    changeLabelVisibleStatus(status) {
      this.addis.customRenderer = status ? CustomRenderer : false;
      this.reloadProcessDesigner();
    },
    elementClick(element) {
      this.element = element;
      // console.log(xmlObj2json(this.xmlString));
      // console.log(this.modeler);
      // if (element.type === "bpmn:UserTask") {
      //   const moddle = window.bpmnInstances.moddle;
      //   const modeling = window.bpmnInstances.modeling;
      //   const child1 = moddle.create("flowable:ChildField", {
      //     id: "child1",
      //     name: "1",
      //     readable: true
      //   });
      //   const child2 = moddle.create("flowable:ChildField", {
      //     id: "child2",
      //     name: "2",
      //     type: "string",
      //     required: true
      //   });
      //   const formProperty = moddle.create("flowable:FormProperty", {
      //     children: [child1, child2]
      //     // children: []
      //   });
      //   const extensionElements = moddle.create("bpmn:ExtensionElements", {
      //     values: [formProperty]
      //   });
      //   modeling.updateProperties(element, {
      //     extensionElements
      //   });
      // }
    },
    requestUserInfo() {
      this.$axios.get("/user/userInfo").then((res) => {
        console.log(res);
      });
    },
  },
};
</script>

<style lang="scss">
body {
  overflow: hidden;
  margin: 0;
  box-sizing: border-box;
}
#app {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  display: inline-grid;
  grid-template-columns: auto max-content;
}
.demo-control-bar {
  position: fixed;
  right: 8px;
  bottom: 8px;
  z-index: 1;
  .open-control-dialog {
    width: 48px;
    height: 48px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 4px;
    font-size: 32px;
    background: rgba(64, 158, 255, 1);
    color: #ffffff;
    cursor: pointer;
  }
}
.control-form {
  .el-radio {
    width: 100%;
    line-height: 32px;
  }
}
body,
body * {
  /* 滚动条 */
  &::-webkit-scrollbar-track-piece {
    background-color: #fff; /*滚动条的背景颜色*/
    -webkit-border-radius: 0; /*滚动条的圆角宽度*/
  }
  &::-webkit-scrollbar {
    width: 10px; /*滚动条的宽度*/
    height: 8px; /*滚动条的高度*/
  }
  &::-webkit-scrollbar-thumb:vertical {
    /*垂直滚动条的样式*/
    height: 50px;
    background-color: rgba(153, 153, 153, 0.5);
    -webkit-border-radius: 4px;
    outline: 2px solid #fff;
    outline-offset: -2px;
    border: 2px solid #fff;
  }
  &::-webkit-scrollbar-thumb {
    /*滚动条的hover样式*/
    background-color: rgba(159, 159, 159, 0.3);
    -webkit-border-radius: 4px;
  }
  &::-webkit-scrollbar-thumb:hover {
    /*滚动条的hover样式*/
    background-color: rgba(159, 159, 159, 0.5);
    -webkit-border-radius: 4px;
  }
}
</style>
