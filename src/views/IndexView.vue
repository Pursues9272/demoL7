<template>
  <div class="index-view">
    <!-- 6<el-button type="primary">Primary</el-button>
    <el-icon><FullScreen /></el-icon> -->
    <div id="map"></div>
    <div class="map-tab">
      <el-checkbox-group v-model="checkList" @change="checkIn">
        <el-checkbox label="1">热力图</el-checkbox>
        <el-checkbox label="2">流线图</el-checkbox>
      </el-checkbox-group>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import { Scene, RasterLayer, HeatmapLayer, WindLayer } from "@antv/l7";
import { Map } from "@antv/l7-maps";
export default {
  data() {
    return {
      checkList: ["1", "2"],
      scene: null,
      layer: null,
    };
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      // 初始化底图
      this.scene = new Scene({
        id: "map",
        logoVisible: false, // logo是否可见
        antialias: true, // 抗锯齿
        map: new Map({
          center: [60, 32.24997445586331],
          zoom: 2,
        }),
      });
      const url =
        "https://t0.tianditu.gov.cn/img_w/wmts?tk=b72aa81ac2b3cae941d1eb213499e15e&";
      const layer = new RasterLayer({
        name: "baseDrawing",
        zIndex: 1,
      }).source(url, {
        parser: {
          type: "rasterTile",
          tileSize: 256,
          wmtsOptions: {
            layer: "img",
            tileMatrixset: "w",
            format: "tiles",
          },
        },
      });
      this.scene.on("loaded", () => {
        this.scene.addLayer(layer);
      });
      this.thermal();
      this.streamline();
    },
    checkIn(value) {
      // 多选框
      let isthermal = false;
      let isstreamline = false;
      for (let i = 0; i < value.length; i++) {
        if (value[i] === "1") {
          isthermal = true;
        } else if (value[i] === "2") {
          this.scene.getLayerByName("streamline").show();
          isstreamline = true;
        }
      }
      if (isthermal) {
        this.scene.getLayerByName("thermal").show();
      } else {
        this.scene.getLayerByName("thermal").hide();
      }

      if (isstreamline) {
        this.scene.getLayerByName("streamline").show();
      } else {
        this.scene.getLayerByName("streamline").hide();
      }
    },
    thermal() {
      // 热力图
      this.scene.on("loaded", () => {
        fetch("./Thermal.json")
          .then((res) => res.json())
          .then((data) => {
            const layer = new HeatmapLayer({
              name: "thermal",
              zIndex: 3,
            })
              .source(data)
              .shape("heatmap")
              .size("mag", [0, 1.0]) // weight映射通道
              .style({
                intensity: 2,
                radius: 20,
                rampColors: {
                  colors: [
                    "#FF4818",
                    "#F7B74A",
                    "#FFF598",
                    "#F27DEB",
                    "#8C1EB2",
                    "#421EB2",
                  ].reverse(),
                  positions: [0, 0.2, 0.4, 0.6, 0.8, 1.0],
                },
              });
            this.scene.addLayer(layer);
          });
      });
    },
    streamline() {
      // 流线图
      this.scene.on("loaded", () => {
        const layer = new WindLayer({
          name: "streamline",
          zIndex: 3,
        });
        layer
          .source(
            "https://gw.alipayobjects.com/mdn/rms_23a451/afts/img/A*wcU8S5xMEDYAAAAAAAAAAAAAARQnAQ",
            {
              parser: {
                type: "image",
                extent: [-360, -85, 360, 85],
              },
            }
          )
          .animate(true)
          .style({
            uMin: -21.32,
            uMax: 26.8,
            vMin: -21.57,
            vMax: 21.42,
            numParticles: 35535,
            fadeOpacity: 0.996,
            sizeScale: 1.2,
            rampColors: {
              0.0: "#3288bd",
              0.1: "#66c2a5",
              0.2: "#abdda4",
              0.3: "#e6f598",
              0.4: "#fee08b",
              0.5: "#fdae61",
              0.6: "#f46d43",
              1.0: "#d53e4f",
            },
          });
        this.scene.addLayer(layer);
      });
    },
  },
};
</script>

<style lang="less" scoped>
// 严格模式下必须存在内容
.index-view {
  width: 100%;
  height: 100%;
  #map {
    width: 100%;
    height: 100%;
    position: absolute;
    left: 0;
    top: 0;
    z-index: 1;
  }
  .map-tab {
    width: 250px;
    height: 50px;
    background-color: rgba(0, 0, 0, 0.8);
    position: absolute;
    top: 0;
    right: 0;
    z-index: 2;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 0 0 0 5px;
    .el-checkbox {
      color: #fff;
    }
  }
}
</style>
