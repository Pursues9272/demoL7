<template>
  <div class="index-view">
    <!-- 6<el-button type="primary">Primary</el-button>
    <el-icon><FullScreen /></el-icon> -->
    <div id="map" ref="mapView"></div>
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
import {
  Scene,
  RasterLayer,
  HeatmapLayer,
  WindLayer,
  CanvasLayer,
} from "@antv/l7";
import { Mapbox } from "@antv/l7-maps";
// impoort * as windy from "@/components/windy";
import Windy from "@/components/windy";
import { Map as Mapmi } from "@arcgis/core/Map";
// import RasterLayer from "../../public/RasterLayer"
// require(["esri/layers/RasterLayer"], function(RasterLayer) { /* code goes here */ });
export default {
  data() {
    return {
      checkList: ["1", "2"],
      scene: null,
      layer: null,
      map: null,
      isWindy: null,
    };
  },
  mounted() {
    this.initMap();
    console.log("Windy=>", Windy);
  },
  methods: {
    initMap() {
      console.log("Mapsaf=>", Map);
      (this.map = new Mapbox({
        style: "./updated.json", // 样式URL
        // center: [120.19382669582967, 30.258134],
        center: [60, 32.24997445586331],
        pitch: 0,
        zoom: 2,
        token:
          "pk.eyJ1IjoicHN5ZHVja2hlYWRhY2hlIiwiYSI6ImNsYXVzeGxydjA3amIzcGsybDR2eGMxbmgifQ.zB-fsUHs1ucH47CsptNOfg",
      })),
        // 初始化底图
        (this.scene = new Scene({
          id: "map",
          logoVisible: false, // logo是否可见
          antialias: true, // 抗锯齿
          map: this.map,
        }));
      // const url =
      //   "https://t0.tianditu.gov.cn/img_w/wmts?tk=b72aa81ac2b3cae941d1eb213499e15e&";
      // const layer = new RasterLayer({
      //   name: "baseDrawing",
      //   zIndex: 1,
      // }).source(url, {
      //   parser: {
      //     type: "rasterTile",
      //     tileSize: 256,
      //     wmtsOptions: {
      //       layer: "img",
      //       tileMatrixset: "w",
      //       format: "tiles",
      //     },
      //   },
      // });
      // layer.on('click', (target) => console.log(target));
      // this.scene.on("loaded", () => {
      //   this.scene.addLayer(layer);
      // });
      // this.scene.map.on('load', () => {
      //   console.log("mapmapmap1=>",this.scene.map)
      // })

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

      const size = this.scene.getSize();
      console.log("size=>", size);

      const width = size[0] + 300;
      const height = size[1] + 200;
      if (isstreamline) {
        this.scene.getLayerByName("streamline").show();
        this.isWindy.start(
          [
            [0, 0],
            [width, height],
          ],
          width,
          height,
          [
            [-257.27912499995625, -5.13222402468837],
            [59.12712499995963, 66.63654745747539],
          ]
        );
      } else {
        this.scene.getLayerByName("streamline").hide();
        this.isWindy.stop();
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
            console.log("mnscene=>", this.scene);
            this.scene.addLayer(layer);
          });
      });
    },
    streamline() {
      // 流线图
      // this.scene.on("loaded", () => {
      //   const layer = new WindLayer({
      //     name: "streamline",
      //     zIndex: 3,
      //   });
      //   layer
      //     .source(
      //       "./streamline.png",
      //       {
      //         parser: {
      //           type: "image",
      //           extent: [-360, -85, 360, 85],
      //         },
      //       }
      //     )
      //     .animate(false)
      //     .style({
      //       uMin: -21.32,
      //       uMax: 26.8,
      //       vMin: -21.57,
      //       vMax: 21.42,
      //       numParticles: 35535,
      //       fadeOpacity: 0.996,
      //       sizeScale: 1.2,
      //       rampColors: {
      //         0.0: "#3288bd",
      //         0.1: "#66c2a5",
      //         0.2: "#abdda4",
      //         0.3: "#e6f598",
      //         0.4: "#fee08b",
      //         0.5: "#fdae61",
      //         0.6: "#f46d43",
      //         1.0: "#d53e4f",
      //       },
      //     });
      //   this.scene.addLayer(layer);
      // });
      // require(["@/components/RasterLayer"],function(
      //   declare, connect, arrayUtils,
      //   domConstruct, domStyle, number,
      //   esriLang, domUtils,
      //   SpatialReference, Point, Layer
      // ) {

      // })
      // console.log("RasterLayer=>",RasterLayer)
      // require(["../../public/RasterLayer"]
      // , function(RasterLayer){
      //   console.log("RasterLayer=>",RasterLayer)
      // }
      // )
      // try{
      //   let RasterLayer = require.config(["./RasterLayer/declare"])
      //   console.log("RasterLayer=>", RasterLayer)
      // }
      // finally {

      // }

      // require(["esri/request"], (esriRequest) => {
      //   console.log("esriRequest=>",esriRequest)
      // })
      // let box = this.$refs.mapView;
      // let canvas = box.getElementsByTagName("canvas");

      this.scene.on("loaded", () => {
        fetch("./gfs.json")
          .then((res) => res.json())
          .then((data) => {
            console.log("data=>", data);
            let _this = this;
            const layer = new CanvasLayer({
              name: "streamline",
              zIndex: 3,
            }).style({
              zIndex: 3,
              update: "dragend",
              opacity: 0.4,
              drawingOnCanvas: (option) => {
                console.log("mapmapmap0=>", this.scene.map);

                // let map = new Mapmi("map", {
                //   center: [-99.076, 39.132],
                //   zoom: 12,
                //   basemap: "dark-gray"
                // });
                // console.log("mapmapmap=>",map)
                // map.on("load", ()=>{
                // console.log("mapmapmap1=>",map.geographicExtent)
                // });

                // require("../../public/RasterLayer",function(RasterLayer){
                // console.log("RasterLayer=>",RasterLayer)
                // })

                const { size, ctx, mapService, canvas } = option;
                const [width, height] = size;
                _this.isWindy = new Windy({ canvas: canvas, data: data });
                console.log("option=>", option);
                console.log("mapService=>", mapService);

                _this.isWindy.stop();
                setTimeout(function () {
                  _this.isWindy.start(
                    [
                      [0, 0],
                      [width, height],
                    ],
                    width,
                    height,
                    [
                      [-257.27912499995625, -5.13222402468837],
                      [59.12712499995963, 66.63654745747539],
                    ]
                    // [[-180, -9.229780544289003],[60, 68.22145106226235]]
                    // [[-180, -85],[60, 32.24997445586331]]
                    // [[extent.xmin, extent.ymin],[extent.xmax, extent.ymax]]
                    // [[0, 0],[120, 32.24997445586331*2]]
                  );
                }, 500);
                // ctx.clearRect(0, 0, width, height);
                // ctx.fillStyle = 'rgba(0, 200, 0, 0.2)';
                // data.features.map(feature => {
                //   console.log("feature=>",feature)
                //   return '';
                // });
                return "";
              },
            });

            console.log("layer=>", layer);
            this.scene.addLayer(layer);

            return "";
          });
        return "";
      });
      console.log("scene", this.scene.map);
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
