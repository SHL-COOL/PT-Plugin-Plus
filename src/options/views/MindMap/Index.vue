<template>
  <div class="container">
    <div class="toolbarContainer">
      <div class="toolbar">
        <div class="toolbarBlock px-3 py-2" >
          <v-btn
              class="toolbarBtn"
              @click="backToRoot"
            >
              复位
              <v-icon>my_location</v-icon>
            </v-btn>

            <v-btn
              class="toolbarBtn"
              @click="toFullscreenShow"
            >
              全屏
              <v-icon>fullscreen</v-icon>
            </v-btn>

            <v-btn
              class="toolbarBtn"
               :loading="exportLoading"
               :disabled="exportLoading"
              @click="exportPng"
            >
              {{ $t('common.share') }}
              <v-icon>share</v-icon>
            </v-btn>
        </div>
      
      </div>
    </div>
    <div id="mindMapContainer" class="mindMapContainer"></div>
  </div>
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
import Export from 'simple-mind-map/src/plugins/Export.js'
// @ts-ignore
import NodeImgAdjust from "simple-mind-map/src/plugins/NodeImgAdjust.js";
// @ts-ignore
import OuterFrame from "simple-mind-map/src/plugins/OuterFrame.js";
// @ts-ignore
import Themes from "simple-mind-map-plugin-themes";
// @ts-ignore
import HandDrawnLikeStyle from './handDrawnLikeStyle.esm.min.js'

/**
 * 思维导图在线编辑
 * https://wanglin2.github.io/mind-map/#/  导入同目录的data.json文件 在线修改后导出 json文件 覆盖同目录data.json
 */
import MapData from "./data.json";

// @ts-ignore
MindMap.usePlugin(TouchEvent)
  .usePlugin(Drag)
  .usePlugin(AssociativeLine)
  .usePlugin(NodeImgAdjust)
  .usePlugin(OuterFrame)
  .usePlugin(Export)
  .usePlugin(HandDrawnLikeStyle);

Themes.init(MindMap);

export default Vue.extend({
  data() {
    return {
      mindMap: null as MindMap | null,
      exportLoading: false,
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
      readonly: true,
      //enableFreeDrag: false,
      //fitPadding: 128
      exportPaddingX: 10,
      exportPaddingY: 10,
      addContentToFooter: () => {
          const el = document.createElement('div')
          el.className = 'footer'
          el.innerHTML = this.$t("app.name").toString()
          const cssText = `
            .footer {
              width: 100%;
              height: 30px;
              display: flex;
              justify-content: center;
              align-items: center;
              font-size: 12px;
              color: #979797;
            }
          `
          return {
            el,
            cssText,
            height: 30
          }
        },
    });

    this.handleShowLoading()
    // @ts-ignore
    if (MapData.root) {
      this.mindMap.setFullData(MapData);
    } else {
      this.mindMap.setData(MapData);
    }
    this.mindMap.view.reset();

    this.mindMap.on('showLoading', this.handleShowLoading);
    this.mindMap.on('node_tree_render_end', this.handleHideLoading);
    window.addEventListener("resize", this.handleResize);
  },
  beforeDestroy() {
    this.mindMap!.off('showLoading', this.handleShowLoading);
    this.mindMap!.off('node_tree_render_end', this.handleHideLoading);
    window.removeEventListener("resize", this.handleResize);
    this.mindMap!.destroy();
  },
  methods: {
    handleResize() {
      this.mindMap!.resize();
    },
    handleShowLoading() {
    },
    handleHideLoading() {
    },   
    enterFullScreen(element : any) {
      if (element.requestFullScreen) {
        element.requestFullScreen()
      } else if (element.webkitRequestFullScreen) {
        element.webkitRequestFullScreen()
      } else if (element.mozRequestFullScreen) {
        element.mozRequestFullScreen()
      }
    },
    toFullscreenShow() {
      this.enterFullScreen(this.mindMap!.el)
    },
    backToRoot() {
      this.mindMap!.renderer.setRootNodeCenter()
    },
    async exportPng() {
      try {
        this.exportLoading = true;
        await this.mindMap!.export('png',true,this.$t("app.name").toString())
        this.exportLoading = false;
      } catch (error) {
        console.log('error:', error);
      }
    },
  }
  },
);
</script>

<style lang="scss" scoped>
.mindMapContainer {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: calc(100vh - 64px - 32px);
}

.toolbarContainer {
  .toolbar {
    position: fixed;
    left: 50%;
    transform: translateX(-50%);
    width: -moz-max-content;
    width: max-content;
    display: flex;
    z-index: 2;
  }


  .toolbarBtn {
    display: flex;
    justify-content: center;
    flex-direction: column;
    cursor: pointer;
    margin-right: 20px;
  }

  .toolbarBlock {
    display: flex;
    border-radius: 6px;
    background-color: #fff;
    box-shadow: 0 2px 16px 0 rgba(0, 0, 0, .06);
    border: 1px solid rgba(0, 0, 0, .06);
    margin-right: 20px;
    flex-shrink: 0;
    position: relative;
}
}

.theme--dark .toolbarContainer {
  .toolbarBlock {
    background-color: #262a2e;
  }
}
</style>