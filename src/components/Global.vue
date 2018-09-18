<template>
  <div class="fm-stretch">
    <div id="cesiumContainer" class="fm-stretch"></div>
    <div class="btn-container" :style="{top: btnPosition.top, left: btnPosition.left}">
      <img src="../../static/images/fuwei.png" @click="flyHome">
      <img :src="abroadBtnImg" @click="showAbroad=!showAbroad">
    </div>
    <div v-if="selectedAbroad">
      <info-box :anchor="center" :type="selectedAbroad.id"></info-box>
    </div>
  </div>
</template>
<script>
import axios from 'axios';
import conf from '../config';
import InfoBox from './InfoBox';

window.CESIUM_BASE_URL = './static/Cesium';
const Cesium = require('cesium/Source/Cesium.js');
require('cesium/Source/Widgets/widgets.css');

// 设置Cesium的默认显示区域，显示出中国全貌
const extent = Cesium.Rectangle.fromDegrees(90, 20, 120, 50);
Cesium.Camera.DEFAULT_VIEW_RECTANGLE = extent;
Cesium.Camera.DEFAULT_VIEW_FACTOR = 1;

export default {
  name: 'global',
  components: {
    InfoBox
  },
  props: ['crowdInfoSource', 'commonInfoSource', 'randomData', 'poiChangedNum'],
  data() {
    return {
      viewer: null,
      randomEntities: [],
      commonEntities: [],
      crowdEntities: [],
      abroadEntities: [], // 所有海外地图的Entiities
      selectedAbroad: null, // 当前选中的海外地图的Entity
      center: [], // 地图中心点的屏幕坐标
      btnPosition: {}, // 控制按钮的显示位置
      showAbroad: false // 是否显示海外地图
    };
  },
  computed: {
    allEntities: function() {
      let arr = [];
      Array.prototype.push.apply(arr, this.commonEntities);
      Array.prototype.push.apply(arr, this.crowdEntities);
      return arr;
    },
    abroadBtnImg() {
      return this.showAbroad ? '../../static/images/abroad.png' : '../../static/images/abroad_normal.png'
    }
  },
  watch: {
    crowdInfoSource: function() {
      if (this.$props.crowdInfoSource) {
        this.loadCrowdData();
      } else {
        for (let i = 0; i < this.crowdEntities.length; i++) {
          this.viewer.entities.remove(this.crowdEntities[i]);
        }
        this.crowdEntities.length = 0;
      }
    },
    commonInfoSource: function() {
      if (this.$props.commonInfoSource) {
        this.loadCommonData();
      } else {
        for (let i = 0; i < this.commonEntities.length; i++) {
          this.viewer.entities.remove(this.commonEntities[i]);
        }
        this.commonEntities.length = 0;
      }
    },
    randomData: function() {
      let c = this.$props.poiChangedNum;
      let i;
      let t = this.allEntities.length - 1;

      for (let m = 0; m < this.randomEntities.length; m++) {
        this.viewer.entities.remove(this.randomEntities[m]);
      }

      this.randomEntities = []
      while (t >= 0 && c > 0) {
        i = parseInt(t * Math.random());
        const pointPosition = this.allEntities[i].position;
        let count = 0;
        const entity = {
          position: pointPosition,
          billboard: {
            image: new Cesium.CallbackProperty(function() {
              // 闪烁次数
              count++

              var tmp = parseInt(count / 2)
              if (tmp >= 34) {
                count = 0
              }
              var image = `./static/billboard/${tmp}.png`
              return image
            }, false),
            width: 32, // default: undefined
            eyeOffset: new Cesium.Cartesian3(0, 0, -500000),
            height: 32
          }
        }

        const entityAdded = this.viewer.entities.add(entity)
        this.randomEntities.push(entityAdded)
        c--;
      }

      // const points = this.generateRandomPoint(10);
      // points.forEach(it => {
      //   this.randomEntities.push(this.viewer.entities.add(it))
      // });
    },
    showAbroad(newVal) {
      if (newVal) {
        this.loadAbroad();
      } else {
        this.clearAbroad();
      }
    }
  },
  methods: {
    reloadData() {
      this.viewer.entities.removeAll();
      this.allEntities.length = 0;
      if (this.$props.crowdInfoSource) {
        this.loadCrowdData();
      }
      if (this.$props.commonInfoSource) {
        setTimeout(() => {
          this.loadCommonData();
        }, 200)
      }
    },
    generateRandomPoint(num) {
      const minLat = 25;
      const maxLat = 45;
      const minLon = 110;
      const maxLon = 120;
      let points = [];
      for (let i = 0; i < num; i++) {
        let cnt = 0;
        const tmpEntity = {
          position: Cesium.Cartesian3.fromDegrees(minLon + (maxLon - minLon) * Math.random(), minLat + (maxLat - minLat) * Math.random()),
          point: {
            pixelSize: new Cesium.CallbackProperty(function() {
              // 闪烁次数
              cnt++;

              var tmp = parseInt(cnt / 2)
              if (tmp >= 26) {
                cnt = 0
              }
              return tmp;
            }, false),
            color: Cesium.Color.TRANSPARENT,
            outlineColor: Cesium.Color.DEEPSKYBLUE,
            outlineWidth: 2
          }
        };
        points.push(tmpEntity);
      }
      return points;
    },
    data2GeoJson(data) {
      let featureCollection = {
        type: "FeatureCollection",
        features: [],
      };
      for (let i = 0; i < data.length; i++) {
        featureCollection.features.push({
          type: 'Feature',
          geometry: {
            type: 'Point',
            coordinates: [data[i].x, data[i].y]
          },
          properties: {}
        })
      }
      return featureCollection;
    },
    loadCrowdData() {
      const that = this;
      axios({
        method: 'get',
        url: conf.serviceUrl + 'statics/crowdInfo'
        // url: 'http://fastmap.navinfo.com/service/statics/crowdInfo',
      }).then(function(res) {
        if (res.data.errcode === 0) {
          let tempSourceData = that.data2GeoJson(res.data.data);
          for (let i = 0; i < tempSourceData.features.length; i++) {
            const feature = tempSourceData.features[i];
            that.crowdEntities.push(that.viewer.entities.add({
              position: Cesium.Cartesian3.fromDegrees(feature.geometry.coordinates[0],
                feature.geometry.coordinates[1]),
              point: {
                pixelSize: 4,
                color: Cesium.Color.fromCssColorString('#fd8e20')
              }
            }));
          }
        }
      }).catch(function(err) {})
    },
    loadCommonData() {
      const that = this;
      axios({
        method: 'get',
        url: conf.serviceUrl + 'statics/commonInfo',
        // url: 'http://fastmap.navinfo.com/service/statics/commonInfo',
        dataType: 'json'
      }).then(function(res) {
        if (res.data.errcode === 0) {
          let tempSourceData = that.data2GeoJson(res.data.data);
          for (let i = 0; i < tempSourceData.features.length; i++) {
            const feature = tempSourceData.features[i];
            that.commonEntities.push(that.viewer.entities.add({
              position: Cesium.Cartesian3.fromDegrees(feature.geometry.coordinates[0],
                feature.geometry.coordinates[1]),
              point: {
                pixelSize: 4,
                color: Cesium.Color.fromCssColorString('#33c3ff')
              }
            }));
          }
        }
      });
    },
    flyHome() {
      this.viewer.camera.flyHome();
    },
    loadAbroad() {
      const abroad = {
        'laowo': [102.36, 17.58],
        'jianpuzhai': [104.55, 11.33]
      };

      for (const [key, val] of Object.entries(abroad)) {
        this.abroadEntities.push(this.viewer.entities.add({
          id: key,
          position: Cesium.Cartesian3.fromDegrees(val[0], val[1]),
          billboard: {
            image: `./static/images/abroad/${key}.png`
          }
        }));
      }

      let lock = false; // 地球在移动动画过程中禁止点击事件
      this.viewer.screenSpaceEventHandler.setInputAction((click) => {
        if (lock) {
          return;
        }

        const scene = this.viewer.scene;
        const camera = this.viewer.camera;
        const canvas = this.viewer.canvas;
        const pickedObject = scene.pick(click.position);
        if (Cesium.defined(pickedObject) && this.abroadEntities.find(it => it.id === pickedObject.id.id)) {
          if (this.selectedAbroad) {
            if (pickedObject.id.id === this.selectedAbroad.id) {
              return;
            }
            this.selectedAbroad.billboard.image = new Cesium.ConstantProperty(`./static/images/abroad/${this.selectedAbroad.id}.png`);
            this.selectedAbroad = null;
          }

          lock = true;
          const height = scene.globe.ellipsoid.cartesianToCartographic(camera.position).height;
          this.viewer.flyTo(pickedObject.id, {
            offset: new Cesium.HeadingPitchRange(camera.heading, camera.pitch, height),
            duration: 1.5
          }).then(() => {
            pickedObject.id.billboard.image = new Cesium.ConstantProperty(`./static/images/abroad/${pickedObject.id.id}_active.png`);
            this.selectedAbroad = pickedObject.id;
            this.center = [canvas.clientWidth / 2, canvas.clientHeight / 2];
            lock = false;
          });
        } else {
          if (this.selectedAbroad) {
            this.selectedAbroad.billboard.image = new Cesium.ConstantProperty(`./static/images/abroad/${this.selectedAbroad.id}.png`);
            this.selectedAbroad = null;
          }
        }
      }, Cesium.ScreenSpaceEventType.LEFT_CLICK);
    },
    clearAbroad() {
      this.abroadEntities.forEach(it => {
        this.viewer.entities.remove(it);
      });
      this.abroadEntities.length = 0;

      this.selectedAbroad = null;

      this.viewer.screenSpaceEventHandler.removeInputAction(Cesium.ScreenSpaceEventType.LEFT_CLICK);
    }
  },
  mounted() {
    const viewer = new Cesium.Viewer('cesiumContainer', {
      sceneMode: Cesium.SceneMode.SCENE3D,
      homeButton: false,
      animation: false,
      timeline: false,
      infoBox: false,
      navigationHelpButton: false,
      baseLayerPicker: false,
      geocoder: false,
      fullscreenButton: false,
      sceneModePicker: false,
      selectionIndicator: false, // 禁用默认的entity选择
      imageryProvider: new Cesium.MapboxImageryProvider({
        mapId: 'mapbox.streets-satellite',
        accessToken: 'pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A',
      }),
      skyBox: new Cesium.SkyBox({
        sources: {
          positiveX: 'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_px.jpg',
          negativeX: 'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_mx.jpg',
          positiveY: 'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_py.jpg',
          negativeY: 'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_my.jpg',
          positiveZ: 'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_pz.jpg',
          negativeZ: 'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_mz.jpg',
        },
      }),
    });

    viewer.extend(Cesium.viewerDragDropMixin);

    // 设置地球初始显示位置
    // const initialPosition = Cesium.Cartesian3.fromDegrees(115.0, 40.69114333714821,
    //   16000000);
    // viewer.scene.camera.setView({
    //   destination: initialPosition,
    //   endTransform: Cesium.Matrix4.IDENTITY,
    // });

    // 禁用默认的鼠标拖动来滚动地图
    // viewer.scene.screenSpaceCameraController.enableRotate = false;

    // 禁用默认的双击一个entity后，自动缩放、定位操作
    viewer.screenSpaceEventHandler.removeInputAction(Cesium.ScreenSpaceEventType.LEFT_DOUBLE_CLICK);
    // 移动开始后清理海外地图的信息框
    viewer.camera.moveStart.addEventListener((moveStartPosition) => {
      if (this.selectedAbroad) {
        this.selectedAbroad.billboard.image = new Cesium.ConstantProperty(`./static/images/abroad/${this.selectedAbroad.id}.png`);
        this.selectedAbroad = null;
      }
    });

    this.viewer = viewer;
    this.reloadData();
    // this.loadAbroad();
    this.btnPosition = {
      top: '180px',
      left: `${this.$el.clientWidth * 0.75 - 90}px`
    };
  }
};

</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#cesiumContainer .cesium-viewer-bottom {
  display: none;
}

.btn-container {
  display: flex;
  align-items: center;
  justify-content: space-evenly;
  justify-content: space-evenly;
  padding: 0px 10px 0px 4px;
  position: absolute;
  width: 128px;
  height: 51px;
  background-image: url(../../static/images/button_frame.png);
  /* 处理被右侧div压盖的问题 */
  z-index: 999;
}

</style>
