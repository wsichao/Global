<template>
  <div id="cesiumContainer"></div>
</template>

<script>
  import axios from 'axios';
  import conf from '../config';

  window.CESIUM_BASE_URL = './static/Cesium';
  const Cesium = require('../../node_modules/cesium/Source/Cesium.js');
  require('../../node_modules/cesium/Source/Widgets/widgets.css');

  export default {
    name: 'global',
    props: ['crowdInfoSource', 'commonInfoSource', 'randomData', 'poiChangedNum'],
    data: function () {
      return {
        randomEntities: [],
        commonEntities: [],
        crowdEntities: []
      };
    },
    computed: {
      allEntities: function () {
        let arr = [];
        Array.prototype.push.apply(arr, this.commonEntities);
        Array.prototype.push.apply(arr, this.crowdEntities);
        return arr;
      }
    },
    watch: {
      crowdInfoSource: function () {
        if (this.$props.crowdInfoSource) {
          this.loadCrowdData();
        } else {
          for (let i=0;i<this.crowdEntities.length;i++) {
            this.viewer.entities.remove(this.crowdEntities[i]);
          }
          this.crowdEntities.length = 0;
        }
      },
      commonInfoSource: function () {
        if (this.$props.commonInfoSource) {
          this.loadCommonData();
        } else {
          for (let i=0;i<this.commonEntities.length;i++) {
            this.viewer.entities.remove(this.commonEntities[i]);
          }
          this.commonEntities.length = 0;
        }
      },
      randomData: function () {
        let c = this.$props.poiChangedNum;
        let i;
        let t = this.allEntities.length - 1;
        
        for(let m = 0; m<this.randomEntities.length; m++){
          this.viewer.entities.remove(this.randomEntities[m]);
        }

        this.randomEntities = []
        while (t >= 0 && c > 0) {
          i = parseInt(t * Math.random());
          let pointPosition = this.allEntities[i].position;
          let count = 0;
          const entity = {
              position : pointPosition,
              billboard : {
                  image : new Cesium.CallbackProperty(function(){
                    // 闪烁次数
                    count++

                    var tmp = parseInt(count/2)
                    if(tmp>=34){
                        count = 0
                    }
                    var image = `../global/static/billboard/${tmp}.png`
                    return image
                  }, false),
                  width : 32, // default: undefined
                  eyeOffset: new Cesium.Cartesian3(0, 0, -500000),
                  height: 32
              }
          }

          const entityAdded = this.viewer.entities.add(entity)
          this.randomEntities.push(entityAdded)
          c--;
        }
      }
    },
    methods: {
      reloadData () {
        this.viewer.entities.removeAll();
        this.allEntities.length = 0;
        if (this.$props.crowdInfoSource){
          this.loadCrowdData();
        }
        const that = this;
        if (this.$props.commonInfoSource){
          setTimeout(function(){
            that.loadCommonData();
          }, 200)
        }
      },
      generateRandomPoint (num) {
        const minLat = 25;
        const maxLat = 45;
        const minLon = 110;
        const maxLon = 120;
        let points = [];
        for (let i = 0; i< num; i++){
          let radius = 7;
          let cnt = 0;
          const tmpEntity = {
            position: Cesium.Cartesian3.fromDegrees(minLon + (maxLon - minLon) * Math.random(), minLat + (maxLat - minLat) * Math.random()),
            point: {
              pixelSize: new Cesium.CallbackProperty(function(){
                let r = radius;
                if (cnt < 5) {
                  if (radius === 7){
                    radius = 14;
                  } else {
                    radius = 7;
                    cnt++;
                  }
                }
                return r;
              }, false),
              color: Cesium.Color.CYAN,
            }
          };
          points.push(tmpEntity);
        }
        return points;
      },
      data2GeoJson (data) {
        let featureCollection={
          type: "FeatureCollection",
          features: [],
        };
        for (let i=0; i<data.length; i++){
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
      loadCrowdData () {
        const that = this;
        axios({
          method: 'get',
          url: conf.serviceUrl + 'statics/crowdInfo'
          // url: 'http://fastmap.navinfo.com/service/statics/crowdInfo',
        }).then(function (res) {
          if (res.data.errcode === 0) {
            let tempSourceData = that.data2GeoJson(res.data.data);
            let i, feature;
            for (i=0; i<tempSourceData.features.length; i++){
              feature = tempSourceData.features[i];
              that.crowdEntities.push(that.viewer.entities.add({
                position: Cesium.Cartesian3.fromDegrees(feature.geometry.coordinates[0],
                        feature.geometry.coordinates[1]),
                point: {
                  pixelSize: 4,
                  color: Cesium.Color.fromCssColorString('#FFCC00')
                }
              }));
            }
          }
        }).catch(function(err){
        })
      },
      loadCommonData () {
        const that = this;
        axios({
          method: 'get',
          url: conf.serviceUrl + 'statics/commonInfo',
          // url: 'http://fastmap.navinfo.com/service/statics/commonInfo',
          dataType: 'json'
        }).then(function (res) {
          if (res.data.errcode === 0) {
            let tempSourceData = that.data2GeoJson(res.data.data);
            let i, feature;
            for (i=0; i<tempSourceData.features.length; i++){
              feature = tempSourceData.features[i];
              that.commonEntities.push(that.viewer.entities.add({
                position: Cesium.Cartesian3.fromDegrees(feature.geometry.coordinates[0],
                        feature.geometry.coordinates[1]),
                point: {
                  pixelSize: 4,
                  color: Cesium.Color.fromCssColorString('#CC3333')
                }
              }));
            }
          }
        }).catch(function(err){
        })
      },
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
          accessToken:
                  'pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4NDg1bDA1cjYzM280NHJ5NzlvNDMifQ.d6e-nNyBDtmQCVwVNivz7A',
        }),
        skyBox: new Cesium.SkyBox({
          sources: {
            positiveX:
                    'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_px.jpg',
            negativeX:
                    'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_mx.jpg',
            positiveY:
                    'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_py.jpg',
            negativeY:
                    'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_my.jpg',
            positiveZ:
                    'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_pz.jpg',
            negativeZ:
                    'static/Cesium/Assets/stars/TychoSkymapII.t3_08192x04096/TychoSkymapII.t3_08192x04096_80_mz.jpg',
          },
        }),
      });

      viewer.extend(Cesium.viewerDragDropMixin);

      // 禁用默认的双击一个entity后，自动缩放、定位操作
      viewer.cesiumWidget.screenSpaceEventHandler.removeInputAction(Cesium.ScreenSpaceEventType.LEFT_DOUBLE_CLICK);

      const initialPosition = Cesium.Cartesian3.fromDegrees(115.0, 40.69114333714821,
              16000000);
      viewer.scene.camera.setView({
        destination: initialPosition,
        endTransform: Cesium.Matrix4.IDENTITY,
      });

      // 禁用默认的鼠标拖动来滚动地图
      // viewer.scene.screenSpaceCameraController.enableRotate = false;

      this.viewer = viewer;
      this.reloadData();
    },

    beforeUpdate() {
      // this.refreshDataSources();
    }
  };
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  #cesiumContainer {
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
  }
  #cesiumContainer .cesium-viewer-bottom {
    display: none;
  }
</style>
