<template>
  <div id="mindMapContainer" class="mindMapContainer"></div>
</template>
  
<script lang="ts">
import Vue from "vue";
import MindMap from "simple-mind-map";
// @ts-ignore
import TouchEvent from "simple-mind-map/src/plugins/TouchEvent.js";
// @ts-ignore
import Drag from "simple-mind-map/src/plugins/Drag.js";
// @ts-ignore
import AssociativeLine from "simple-mind-map/src/plugins/AssociativeLine.js";
// @ts-ignore
import NodeImgAdjust from "simple-mind-map/src/plugins/NodeImgAdjust.js";
// @ts-ignore
import OuterFrame from "simple-mind-map/src/plugins/OuterFrame.js";
// @ts-ignore
import Themes from "simple-mind-map-plugin-themes";

/**
 * 思维导图数据源
 * https://wanglin2.github.io/mind-map/#/  导入同目录的data.json文件 在线修改后导出 json文件 覆盖同目录data.json
 */
import MapData from "./data.json";

// @ts-ignore
MindMap.usePlugin(TouchEvent)
  .usePlugin(Drag)
  .usePlugin(AssociativeLine)
  .usePlugin(NodeImgAdjust)
  .usePlugin(OuterFrame);

Themes.init(MindMap);

export default Vue.extend({
  data() {
    return {
      mindMap: null as MindMap | null,
    };
  },
  mounted() {
    // @ts-ignore
    this.mindMap = new MindMap({
      el: document.getElementById("mindMapContainer"),
      tagPosition: "bottom",
      fit: true,
      mousewheelAction: "zoom",
      emptyTextMeasureHeightText: "",
      textAutoWrapWidth: 1000,
      enableFreeDrag: false,
      fitPadding: 128
    });

    console.log(this.mindMap);

    // @ts-ignore
    if (MapData.root) {
      this.mindMap.setFullData(MapData);
    } else {
      this.mindMap.setData(MapData);
    }
    this.mindMap.view.reset();
    window.addEventListener("resize", this.handleResize);
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.handleResize);
    this.mindMap!.destroy();
  },
  methods: {
    handleResize() {
      this.mindMap!.resize();
    },
  },
});
</script>
  
<style lang="scss" scoped>
.mindMapContainer {
  width: 100%;
  height: calc(100vh - 64px - 64px);
}
</style>
  