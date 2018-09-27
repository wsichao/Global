<template>
  <div class="fm-stretch">
    <div id="cesiumContainer" class="fm-stretch"></div>
    <div v-if="selectedAbroadEntity">
      <abroad-info :anchor="center" :info="selectedAbroad"></abroad-info>
    </div>
  </div>
</template>
<script>
import axios from 'axios';
import conf from '../config';
import AbroadInfo from './AbroadInfo';

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
    AbroadInfo
  },
  props: ['showCrowd', 'showCommon', 'showBase', 'showAbroad', 'poiChangedNum', 'reset'],
  data() {
    return {
      viewer: null,
      commonEntities: [],
      commonRandomEntities: [],
      crowdEntities: [],
      crowdRandomEntities: [],
      baseEntities: [],
      abroadEntities: [], // 所有海外地图的Entiities
      abroadList: [],
      selectedAbroadEntity: null, // 当前选中的海外地图的Entity
      center: [] // 地图中心点的屏幕坐标
    };
  },
  computed: {
    selectedAbroad() {
      return this.abroadList.find(it => it.id === this.selectedAbroadEntity.id);
    }
  },
  watch: {
    showCrowd: function(newVal) {
      if (newVal) {
        this.loadCrowdData();
      } else {
        this.clearSpecEntities(this.crowdEntities);
        this.clearSpecEntities(this.crowdRandomEntities);
      }
    },
    showCommon: function(newVal) {
      if (newVal) {
        this.loadCommonData();
      } else {
        this.clearSpecEntities(this.commonEntities);
        this.clearSpecEntities(this.commonRandomEntities);
      }
    },
    poiChangedNum: function(newVal) {
      this.clearSpecEntities(this.commonRandomEntities);
      this.clearSpecEntities(this.crowdRandomEntities);

      let c = newVal;
      const allEntities = [...this.commonEntities, ...this.crowdEntities];
      const t = allEntities.length - 1;
      while (t >= 0 && c > 0) {
        const i = parseInt(t * Math.random());
        const orginEntity = allEntities[i];
        let count = 0;
        const entity = {
          position: orginEntity.position,
          billboard: {
            image: new Cesium.CallbackProperty(function() {
              // 闪烁次数
              count++;
              let tmp = parseInt(count / 2);
              if (tmp >= 49) {
                count = 0;
              }
              tmp = String(tmp).padStart(5, '0');
              let name = orginEntity.name === 'crowd' ? 'blue' : 'yellow';
              return `./static/billboard2/${name}_${tmp}.png`;
            }, false),
            // width: 32, // default: undefined
            // eyeOffset: new Cesium.Cartesian3(0, 0, -500000),
            // height: 32
          }
        }

        const entityAdded = this.viewer.entities.add(entity)
        if (orginEntity.name === 'crowd') {
          this.crowdRandomEntities.push(entityAdded);
        } else {
          this.commonRandomEntities.push(entityAdded);
        }
        c--;
      }

      // const points = this.generateRandomPoint(10);
      // points.forEach(it => {
      //   this.randomEntities.push(this.viewer.entities.add(it))
      // });
    },
    showBase(newVal) {
      if (newVal) {
        this.loadBase();
      } else {
        this.clearSpecEntities(this.baseEntities);
      }
    },
    showAbroad(newVal) {
      if (newVal) {
        this.loadAbroad();
      } else {
        this.clearSpecEntities(this.abroadEntities);

        this.abroadList = [];
        this.selectedAbroadEntity = null;
        this.viewer.screenSpaceEventHandler.removeInputAction(Cesium.ScreenSpaceEventType.LEFT_CLICK);
      }
    },
    reset() {
      this.flyHome();
    }
  },
  methods: {
    loadData() {
      this.viewer.entities.removeAll();
      if (this.showCrowd) {
        this.loadCrowdData();
      }
      if (this.showCommon) {
        this.loadCommonData();
      }
      if (this.showBase) {
        this.loadBase();
      }
      if (this.showAbroad) {
        this.loadAbroad();
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
            outlineWidth: 4
          }
        };
        points.push(tmpEntity);
      }
      return points;
    },
    data2GeoJson(data) {
      let featureCollection = {
        type: 'FeatureCollection',
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
      }).then(function(res) {
        if (res.data.errcode === 0) {
          let tempSourceData = that.data2GeoJson(res.data.data);
          for (let i = 0; i < tempSourceData.features.length; i++) {
            const feature = tempSourceData.features[i];
            that.crowdEntities.push(that.viewer.entities.add({
              name: 'crowd',
              position: Cesium.Cartesian3.fromDegrees(feature.geometry.coordinates[0],
                feature.geometry.coordinates[1]),
              point: {
                
                pixelSize: 3,
                color: Cesium.Color.fromCssColorString('#3493ff')
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
        dataType: 'json'
      }).then(function(res) {
        if (res.data.errcode === 0) {
          let tempSourceData = that.data2GeoJson(res.data.data);
          for (let i = 0; i < tempSourceData.features.length; i++) {
            const feature = tempSourceData.features[i];
            that.commonEntities.push(that.viewer.entities.add({
              name: 'common',
              position: Cesium.Cartesian3.fromDegrees(feature.geometry.coordinates[0],
                feature.geometry.coordinates[1]),
              point: {
                pixelSize: 3,
                color: Cesium.Color.fromCssColorString('#fd8e2a')
              }
            }));
          }
        }
      });
    },
    flyHome() {
      this.viewer.camera.flyHome(1.5);
    },
    loadBase() {
      axios({
        method: 'get',
        url: './static/waiyejidi.json',
        dataType: 'json'
      }).then(res => {
        if (res.data) {
          res.data.forEach(item => {
            let offsetY = 0;
            if (item.type === 1) {
              offsetY = -17;
            } else if (item.type === 2) {
              offsetY = -14;
            } else if (item.type === 3) {
              offsetY = -11;
            }
            this.baseEntities.push(this.viewer.entities.add({
              id: `base-${item.id}`,
              position: Cesium.Cartesian3.fromDegrees(item.lonlat[0],
                item.lonlat[1]),
              billboard: {
                image: `./static/images/base_type_${item.type}.png`,
                pixelOffset: new Cesium.Cartesian2(0, offsetY)
              }
            }));
          });
        }
      });
    },
    clearSelectedAbroadEntity() {
      this.selectedAbroadEntity.billboard.image = new Cesium.ConstantProperty(`./static/images/abroad/${this.selectedAbroadEntity.id}.png`);
      this.selectedAbroadEntity = null;
    },
    bindAbroadEvent() {
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
          const pickedEntity = pickedObject.id;
          if (this.selectedAbroadEntity) {
            if (pickedEntity.id === this.selectedAbroadEntity.id) {
              return;
            }
            this.clearSelectedAbroadEntity();
          }

          lock = true;
          const height = scene.globe.ellipsoid.cartesianToCartographic(camera.position).height;
          this.viewer.flyTo(pickedEntity, {
            offset: new Cesium.HeadingPitchRange(camera.heading, camera.pitch, height),
            duration: 1.5
          }).then(() => {
            pickedEntity.billboard.image = new Cesium.ConstantProperty(`./static/images/abroad/${pickedEntity.id}_active.png`);
            this.selectedAbroadEntity = pickedEntity;
            this.center = [canvas.clientWidth / 2, canvas.clientHeight / 2];
            lock = false;
          });
        } else {
          if (this.selectedAbroadEntity) {
            this.clearSelectedAbroadEntity();
          }
        }
      }, Cesium.ScreenSpaceEventType.LEFT_CLICK);
    },
    loadAbroad() {
      axios({
        method: 'get',
        url: './static/haiwai.json',
        dataType: 'json'
      }).then(res => {
        if (res.data) {
          this.abroadList = res.data;
          this.abroadList.forEach(item => {
            this.abroadEntities.push(this.viewer.entities.add({
              id: item.id,
              position: Cesium.Cartesian3.fromDegrees(item.lonlat[0],
                item.lonlat[1]),
              billboard: {
                image: `./static/images/abroad/${item.id}.png`
              }
            }));
          });

          this.bindAbroadEvent();
        }
      });
    },
    clearSpecEntities(entitiesArray) {
      entitiesArray.forEach(it => {
        this.viewer.entities.remove(it);
      });
      entitiesArray.length = 0;
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
    viewer.screenSpaceEventHandler.setInputAction(() => {
      this.flyHome();
    }, Cesium.ScreenSpaceEventType.LEFT_DOUBLE_CLICK);
    // 移动开始后清理海外地图的信息框
    viewer.camera.moveStart.addEventListener((moveStartPosition) => {
      if (this.selectedAbroadEntity) {
        this.clearSelectedAbroadEntity();
      }
    });

    this.viewer = viewer;
    this.loadData();
  }
};

</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>


</style>
