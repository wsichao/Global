<template>
  <div class="fm-stretch">
    <Global :poi-changed-num="mapData.randomPoiChangedNum" :show-crowd="mapData.showCrowd" :show-common="mapData.showCommon" :show-base="mapData.showBase" :show-abroad="mapData.showAbroad" :reset="mapData.reset"></Global>
    <div class="fm-stretch flex-layout float">
      <div class="col flex-layout-v" style="width: 25%;align-items: flex-start;">
        <div class="top-title left">
          <div class="text">
            <img src="./assets/icon_2.png">
            <div style="padding:0 10px">数据来源</div>
          </div>
        </div>
        <div class="flex-layout-v" style="height: 100%;padding-left: 40px;">
          <panel title="自采 Field collection">
            <div slot="content">
              <div>
                <bar-road-chart :roadData='charData.road'></bar-road-chart>
              </div>
              <div>
                <bar-poi-chart :poiData='charData.poi'></bar-poi-chart>
              </div>
            </div>
          </panel>
          <panel title="第三方数据 Third party data">
            <div slot="content">
              <line-chart :chartData='charData.thrid'></line-chart>
            </div>
          </panel>
          <panel title="众包 Crowdsourcing">
            <div slot="content">
              <line-chart-crowd :chartData='crowd'></line-chart-crowd>
            </div>
          </panel>
        </div>
        <div class="bottom-title left">
          <div class="text">
            Data Sources
          </div>
        </div>
      </div>
      <div class="flex-layout-v fm-stretch" style="width: 40%;align-items: center;">
        <div class="center-content">
          <div style="padding: 20px 130px;">
            <div>
              <banner></banner>
            </div>
            <div class="flex-layout summary">
              <div>
                <div>道路总量:<span class="num-text">{{title.roadLen | splitSymbol}}</span>公里</div>
                <div>道路更新:<span class="num-text">{{title.perUpdateRoadLable | splitSymbol}}</span> 公里</div>
                <div>道路新增:<span class="num-text">{{title.perAddRoadLable | splitSymbol}}</span> 公里</div>
              </div>
              <div>
                <div>POI总量:<span class="num-text">{{title.poiNum | splitSymbol}}</span>个</div>
                <div>POI更新:<span class="num-text">{{title.perUpdatePoiLable | splitSymbol}}</span>个</div>
                <div>POI新增:<span class="num-text">{{title.perAddPoiLable | splitSymbol}}</span>个</div>
              </div>
            </div>
          </div>
        </div>
        <div class="legendContainer">
          <img src="../static/images/fuwei.png" class="img-btn" @click="mapData.reset=Math.random()">
          <div class="legend">
            <div @click="mapData.showCommon=!mapData.showCommon">
              <span :style="{'background-color': mapData.showCommon ? '#fd8e2a' : '#9da0a4'}"></span> 自采分布
            </div>
            <div @click="mapData.showCrowd=!mapData.showCrowd">
              <span :style="{'background-color': mapData.showCrowd ? '#3493ff' : '#9da0a4'}"></span> 众包分布
            </div>
            <div @click="mapData.showBase=!mapData.showBase">
              <span :style="{'background-color': mapData.showBase ? '#0b8415' : '#9da0a4'}"></span> 外业基地
            </div>
          </div>
          <img :src="abroadBtnImg" class="img-btn" @click="mapData.showAbroad=!mapData.showAbroad">
        </div>
      </div>
      <div class="col flex-layout-v" style="width:25%;align-items: flex-end">
        <div class="top-title right" style="padding-top: 0%;align-items: flex-start;">
          <div class="text">
            <div style="padding:0 10px">数据发布</div>
            <img src="./assets/icon_1.png">
          </div>
        </div>
        <div class="flex-layout-v" style="height: 100%;padding-right: 40px;">
          <panel title="日出品 Daily Release">
            <div slot="content">
              <day-chart :dayProduce="charData.dayProduce"></day-chart>
            </div>
          </panel>
          <panel title="月出品 Monthly Release">
            <div slot="content">
              <month-chart :monthProduce="charData.monthProduce"></month-chart>
            </div>
          </panel>
          <panel title="季出品 Quarterly Release" :sub-title="season.spVerson">
            <div slot="content" class="flex-layout">
              <dashboard-chart :dashboard="season.road"></dashboard-chart>
              <dashboard-chart :dashboard="season.poi"></dashboard-chart>
            </div>
          </panel>
        </div>
        <div class="bottom-title right">
          <div class="text">
            Data Release
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import BarPoiChart from "@/components/chart/BarPoiChart";
import BarRoadChart from "@/components/chart/BarRoadChart";
import LineChart from "@/components/chart/LineChart";
import LineChartCrowd from "@/components/chart/LineChartCrowd";
import DashboardChart from "@/components/chart/DashboardChart";
import DayChart from "@/components/chart/DayChart";
import MonthChart from "@/components/chart/MonthChart";
import Banner from "@/components/Banner";
import Global from "@/components/Global";
import SplitNumber from "@/components/SplitNumber";
import axios from "axios";
import Panel from "@/components/Panel";
import conf from "./config";

const TWEEN = require("@tweenjs/tween.js");

export default {
  name: "app",
  data() {
    return {
      title: {
        perUpdateRoad: 0,
        perAddRoad: 0,
        perUpdatePoi: 0,
        perAddPoi: 0,
        perAddRoadLable: 0,
        perUpdateRoadLable: 0,
        perAddPoiLable: 0,
        perUpdatePoiLable: 0
      },
      crowd: {
        lineData: []
      },
      season: {
        road: {},
        poi: {}
      },
      charData: {
        poi: {
          newData: [],
          updateData: [],
          xAxis: [],
          cUpdatePoi: 0,
          cAddPoi: 0
        },
        road: {
          newData: [],
          updateData: [],
          xAxis: [],
          cUpdateRoad: 0,
          cAddRoad: 0
        },
        dayProduce: {
          barData: [],
          lineData: [],
          yAxis: []
        },
        monthProduce: {
          barData: [],
          lineData: [],
          yAxis: []
        },
        thrid: {
          lineData: [
            [],
            [],
            []
          ], //  用户轨迹点,用户问题反馈,互联网信息
          xAxis: []
        }
      },
      mapData: {
        showCommon: true,
        showCrowd: true,
        showBase: true,
        showAbroad: false,
        randomPoiChangedNum: 5 + parseInt(20 * Math.random()),
        reset: 0
      },
      timeout: null,
      tweenColor: ""
    };
  },
  computed: {
    abroadBtnImg() {
      return this.mapData.showAbroad ? '../static/images/abroad.png' : '../static/images/abroad_normal.png'
    },
  },
  filters: {
    splitSymbol(value) {
      // 将数字三位隔开
      return parseInt(value).toLocaleString();
    }
  },
  watch: {
    // 用于统计信息的过度效果显示
    "title.perAddRoad": {
      handler(newValue, oldValue) {
        this.tweenChange("perAddRoadLable", newValue, oldValue);
      }
    },
    "title.perUpdateRoad": {
      handler(newValue, oldValue) {
        this.tweenChange("perUpdateRoadLable", newValue, oldValue);
      }
    },
    "title.perAddPoi": {
      handler(newValue, oldValue) {
        this.tweenChange("perAddPoiLable", newValue, oldValue);
      }
    },
    "title.perUpdatePoi": {
      handler(newValue, oldValue) {
        this.tweenChange("perUpdatePoiLable", newValue, oldValue);
      }
    }
  },
  methods: {
    tweenChange(lable, newValue, oldValue) {
      // 用于统计信息的过度效果显示
      let vm = this;

      function animate() {
        if (TWEEN.update()) {
          requestAnimationFrame(animate);
        }
      }
      let t = new TWEEN.Tween({
          tweeningNumber: oldValue
        })
        .easing(TWEEN.Easing.Quadratic.Out)
        .to({
            tweeningNumber: newValue
          },
          1000 * 2
        ) // 数据变化持续2秒
        .onUpdate(function() {
          vm.title[lable] = this.tweeningNumber.toFixed(0);
          vm.tweenColor = "#fff";
        })
        .onComplete(function() {
          vm.tweenColor = "";
        })
        .start();
      animate();
    },
    getChartData() {
      const that = this;
      axios({
          method: "get",
          url: conf.serviceUrl + "statics/productMonitor"
          // url: 'http://fastmap.navinfo.com/18spr/service/statics/productMonitor',
          // url: 'http://fastmap.navinfo.com/service/statics/productMonitor',
        })
        .then(function(res) {
          if (res && res.data.errcode == 0) {
            that.recomData(res.data.data);
          }
        })
        .catch(function(err) {});
    },
    recomData(data) {
      // 格式化接口数据
      // 临时的假数据
      // data.perAddPoi = 12616;
      // data.perUpdatePoi = 35418;
      // data.perAddRoad = 1432;
      // data.perUpdateRoad = 6345;

      this.initOriginData(data);
      this.titleData(data);
      this.recomPoi(data);
      this.recomRoad(data);
      this.recomDayProduce(data);
      this.recomMonthProduce(data);
      this.recomThird(data);
      this.crowdData(data);
      this.seasonData(data);
    },
    seasonData(data) {
      this.season.road = {
        cahrtId: "roadDash",
        titleName: "道路",
        styleColor: "rgba(51,153,255,0.5)",
        dataUnit: "公里",
        dataUpdate: data.spUpdateRoad,
        dataAdd: data.spAddRoad
      };

      this.season.poi = {
        cahrtId: "poiDash",
        titleName: "POI",
        styleColor: "rgba(253,142,32,0.5)",
        dataUnit: "个",
        dataUpdate: data.spUpdatePoi,
        dataAdd: data.spAddPoi
      };
      this.season.spVerson = data.spVerson;
    },
    crowdData(data) {
      // 众包
      this.crowd.crowdUserNum = data.crowdUserNum;
      this.crowd.crowdRoadLen = data.crowdRoadLen;
      this.crowd.crowdPoiNum = data.crowdPoiNum;

      let months = Object.keys(data.crowdEachMonth).sort((a, b) => {
        return a - b;
      });
      let lineData = [
          [],
          []
        ],
        xAxis = [];
      months.forEach(e => {
        lineData[0].push(data.crowdEachMonth[Number(e)].road);
        lineData[1].push(data.crowdEachMonth[Number(e)].poi);
        xAxis.push(e + "月");
      });

      this.crowd.lineData = lineData;
      this.crowd.xAxis = xAxis;
    },
    getCurrentProcess(filed, data) {
      let currentHour = new Date().getHours() - 8; // 当前小时
      let step = Math.ceil(data[filed] / 15 * currentHour);
      if (step > data[filed]) {
        step = data[filed];
      }
      return step;
    },
    titleData(data) {
      let times = 60 * 15 * 2; // 总共更新的次数 （一分钟刷新一次，24小时刷新60*24次，可以根据具体效果设置时长）
      let that = this;
      if (this.timeout) {
        clearTimeout(this.timeout);
      }

      let _refreshData = function() {
        let improve1 = that.updatePerAddRoad(times, data);
        let improve2 = that.updatePerUpdateRoad(times, data);
        let improve3 = that.updatePerAddPoi(times, data); // poi新增每次递增值
        let improve4 = that.updatePerUpdatePoi(times, data); // poi修改每次递增值

        that.title.roadLen = that.title.roadLen + improve1;
        that.title.poiNum = that.title.poiNum + improve3;
        that.mapData.randomTaskNum = Math.random();
        // that.mapData.randomPoiChangedNum = improve3 + improve4;
        that.mapData.randomPoiChangedNum = 5 + parseInt(20 * Math.random());
        let currentMonth = new Date().getMonth() + 1;
        data.cRoadAverage[currentMonth].add =
          data.cRoadAverage[currentMonth].add + improve1;
        data.cRoadAverage[currentMonth].update =
          data.cRoadAverage[currentMonth].update + improve2;
        data.cAddRoad = data.cAddRoad + improve1;
        data.cUpdateRoad = data.cUpdateRoad + improve2;
        data.cPoiAverage[currentMonth].add =
          data.cPoiAverage[currentMonth].add + improve3;
        data.cPoiAverage[currentMonth].update =
          data.cPoiAverage[currentMonth].update + improve4;
        data.cAddPoi = data.cAddPoi + improve3;
        data.cUpdatePoi = data.cUpdatePoi + improve4;
        that.recomPoi(data);
        that.recomRoad(data);
      };

      let _startTimer = function() {
        const timer = (10 + parseInt(20 * Math.random())) * 1000;
        that.timeout = setTimeout(function() {
          _refreshData();
          _startTimer();
        }, timer);
      };

      _startTimer();
    },
    updatePerAddRoad(times, data) {
      let addStep = 0;
      // let step = Math.ceil(data.perAddRoad / times);
      let step = parseInt(3 * Math.random());
      if (this.title.perAddRoad + step < data.perAddRoad) {
        this.title.perAddRoad = this.title.perAddRoad + step;
        addStep = step;
      } else {
        addStep = data.perAddRoad - this.title.perAddRoad;
        this.title.perAddRoad = data.perAddRoad;
      }
      return addStep;
    },
    updatePerUpdateRoad(times, data) {
      let addStep = 0;
      // let step = Math.ceil(data.perUpdateRoad / times);
      let step = parseInt(5 * Math.random());
      if (this.title.perUpdateRoad + step < data.perUpdateRoad) {
        this.title.perUpdateRoad = this.title.perUpdateRoad + step;
        addStep = step;
      } else {
        addStep = data.perUpdateRoad - this.title.perUpdateRoad;
        this.title.perUpdateRoad = data.perUpdateRoad;
      }
      return addStep;
    },
    updatePerAddPoi(times, data) {
      let addStep = 0;
      // let step = Math.ceil(data.perAddPoi / times);
      let step = 10 + parseInt(10 * Math.random());
      if (this.title.perAddPoi + step < data.perAddPoi) {
        this.title.perAddPoi = this.title.perAddPoi + step;
        addStep = step;
      } else {
        addStep = data.perAddPoi - this.title.perAddPoi;
        this.title.perAddPoi = data.perAddPoi;
      }
      return addStep;
    },
    updatePerUpdatePoi(times, data) {
      let addStep = 0;
      // let step = Math.ceil(data.perUpdatePoi / times);
      let step = 20 + parseInt(10 * Math.random());
      if (this.title.perUpdatePoi + step < data.perUpdatePoi) {
        this.title.perUpdatePoi = this.title.perUpdatePoi + step;
        addStep = step;
      } else {
        addStep = data.perUpdatePoi - this.title.perUpdatePoi;
        this.title.perUpdatePoi = data.perUpdatePoi;
      }
      return addStep;
    },
    initOriginData(data) {
      // 根据当前的时间设置初始值
      this.title.perUpdateRoad = this.getCurrentProcess("perUpdateRoad", data);
      this.title.perAddRoad = this.getCurrentProcess("perAddRoad", data);
      this.title.perUpdatePoi = this.getCurrentProcess("perUpdatePoi", data);
      this.title.perAddPoi = this.getCurrentProcess("perAddPoi", data);
      // 重置总量 poi和道路的界面显示值 等于 总数量-今日新增-今日更新+新增初始值+更新初始值
      data.roadLen = data.roadLen - data.perAddRoad + this.title.perAddRoad;
      data.poiNum = data.poiNum - data.perAddPoi + this.title.perAddPoi;
      this.title.roadLen = data.roadLen;
      this.title.poiNum = data.poiNum;
      // 初始化自采道路图表
      let currentMonth = new Date().getMonth() + 1;
      data.cRoadAverage[currentMonth].add =
        data.cRoadAverage[currentMonth].add -
        data.perAddRoad +
        this.title.perAddRoad;
      data.cRoadAverage[currentMonth].update =
        data.cRoadAverage[currentMonth].update -
        data.perUpdateRoad +
        this.title.perUpdateRoad;
      data.cAddRoad = data.cAddRoad - data.perAddRoad + this.title.perAddRoad;
      data.cUpdateRoad =
        data.cUpdateRoad - data.perUpdateRoad + this.title.perUpdateRoad;
      data.cPoiAverage[currentMonth].add =
        data.cPoiAverage[currentMonth].add -
        data.perAddPoi +
        this.title.perAddPoi;
      data.cPoiAverage[currentMonth].update =
        data.cPoiAverage[currentMonth].update -
        data.perUpdatePoi +
        this.title.perUpdatePoi;
      data.cAddPoi = data.cAddPoi - data.perAddPoi + this.title.perAddPoi;
      data.cUpdatePoi =
        data.cUpdatePoi - data.perUpdatePoi + this.title.perUpdatePoi;
    },
    recomPoi(data) {
      // 重组poi数据,使之符合图表格式
      const poiAvg = data.cPoiAverage;
      // this.charData.poi.cUpdatePoi = data.cUpdatePoi;
      // this.charData.poi.cAddPoi = data.cAddPoi;
      this.charData.poi.newData = [];
      this.charData.poi.updateData = [];
      this.charData.poi.xAxis = [];
      let cAdd = 0;
      let cUpdate = 0;
      for (let i in poiAvg) {
        if (poiAvg.hasOwnProperty(i)) {
          this.charData.poi.xAxis.push(i + "月");
          this.charData.poi.newData.push(poiAvg[i].add);
          this.charData.poi.updateData.push(poiAvg[i].update);
          cAdd += poiAvg[i].add;
          cUpdate += poiAvg[i].update;
        }
      }
      this.charData.poi.cUpdatePoi = cUpdate;
      this.charData.poi.cAddPoi = cAdd;
    },
    recomRoad(data) {
      // 重组road数据,使之符合图表格式
      let roadAvg = data.cRoadAverage;
      // this.charData.road.cUpdateRoad = data.cUpdateRoad;
      // this.charData.road.cAddRoad = data.cAddRoad;
      this.charData.road.newData = [];
      this.charData.road.updateData = [];
      this.charData.road.xAxis = [];
      let cAdd = 0;
      let cUpdate = 0;
      for (let i in roadAvg) {
        if (roadAvg.hasOwnProperty(i)) {
          this.charData.road.xAxis.push(i + "月");
          this.charData.road.newData.push(roadAvg[i].add);
          this.charData.road.updateData.push(roadAvg[i].update);
          cAdd += roadAvg[i].add;
          cUpdate += roadAvg[i].update;
        }
      }
      this.charData.road.cUpdateRoad = cUpdate;
      this.charData.road.cAddRoad = cAdd;
    },
    recomThird(data) {
      // 重组road数据,使之符合图表格式
      let inforDetail = data.thirdInforDetail; // 用户轨迹点 -- 数据情报收集详情
      let userDetail = data.thirdUserDetail; // 用户问题反馈  -- 用户反馈详情
      let webDetail = data.thirdWebDetail; // 互联网信息  -- 互联网信息详情
      let inforTotal = data.thirdInforTotal; // 用户轨迹点 -- 数据情报收集
      let userTotal = data.thirdUserTotal; // 用户问题反馈  -- 用户反馈
      let webTotal = data.thirdWebTotal; // 互联网信息  -- 互联网信息
      this.charData.thrid.inforTotal = inforTotal;
      this.charData.thrid.userTotal = userTotal;
      this.charData.thrid.webTotal = webTotal;
      this.charData.thrid.lineData = [
        [],
        [],
        []
      ];
      this.charData.thrid.xAxis = [];
      for (let i in inforDetail) {
        if (inforDetail.hasOwnProperty(i)) {
          this.charData.thrid.xAxis.push(i + "月");
          this.charData.thrid.lineData[0].push(inforDetail[i]);
          this.charData.thrid.lineData[1].push(userDetail[i]);
          this.charData.thrid.lineData[2].push(webDetail[i]);
        }
      }
    },

    recomDayProduce(data) {
      // 重组road数据,使之符合图表格式
      // add by chenx on 2018-9-10 for story#6678
      data.dpAddPoi = Math.ceil(data.dpAddPoi / 1000);
      data.dpUpdatePoi = Math.ceil(data.dpUpdatePoi / 1000);
      data.dpAverage.addPoi = Math.ceil(data.dpAverage.addPoi / 1000);
      data.dpAverage.updatePoi = Math.ceil(data.dpAverage.updatePoi / 1000);
      this.charData.dayProduce.barData = [
        data.dpAddPoi,
        data.dpUpdatePoi,
        data.dpAddRoad,
        data.dpUpdateRoad
      ];
      this.charData.dayProduce.lineData = [
        data.dpAverage.addPoi,
        data.dpAverage.updatePoi,
        data.dpAverage.addRoad,
        data.dpAverage.updateRoad
      ];
      // this.charData.dayProduce.yAxis = [`新增POI  （个）`, `更新POI  （个）`, `新增道路  （公里）`, `更新道路  （公里）`];
      this.charData.dayProduce.yAxis = [
        "新增POI",
        "更新POI",
        "新增道路",
        "更新道路"
      ];
      this.charData.dayProduce.unit = [
        "个",
        "个",
        "公里",
        "公里"
      ];
      //${data.dpAddPoi}\\${data.dpUpdateRoad}\\${data.dpAddRoad}\\${data.dpUpdateRoad}
    },
    recomMonthProduce(data) {
      // 重组road数据,使之符合图表格式
      // add by chenx on 2018-9-10 for story#6678
      data.mpAddPoi = Math.ceil(data.mpAddPoi);
      data.mpUpdatePoi = Math.ceil(data.mpUpdatePoi);
      data.mpAverage.addPoi = Math.ceil(data.mpAverage.addPoi);
      data.mpAverage.updatePoi = Math.ceil(data.mpAverage.updatePoi);
      this.charData.monthProduce.barData = [
        data.mpAddPoi,
        data.mpUpdatePoi,
        data.mpAddRoad,
        data.mpUpdateRoad
      ];
      this.charData.monthProduce.lineData = [
        data.mpAverage.addPoi,
        data.mpAverage.updatePoi,
        data.mpAverage.addRoad,
        data.mpAverage.updateRoad
      ];
      // this.charData.monthProduce.yAxis = [`新增POI`, `更新POI  （个）`, `新增道路  （公里）`, `更新道路  （公里）`];
      this.charData.monthProduce.yAxis = [
        "新增POI",
        "更新POI",
        "新增道路",
        "更新道路"
      ];
      this.charData.monthProduce.unit = [
        "个",
        "个",
        "公里",
        "公里"
      ];
      //${data.mpAddPoi} 个${data.mpUpdatePoi} 个${data.mpAddRoad} 公里${data.mpUpdateRoad} 公里
    },
    toggleDataSource(type) {
      if (type === "common") {
        this.mapData.showCommon = !this.mapData.showCommon;
      } else {
        this.mapData.showCrowd = !this.mapData.showCrowd;
      }
    }
  },
  created() {
    this.getChartData();
    setInterval(function() {
      var date = new Date();
      if (date.getHours() == 23) {
        // 每天23点执行一次
        this.getChartData();
      }
    }, 1000 * 60 * 60);
  },
  beforeDestroy() {
    clearTimeout(this.timeout);
  },
  components: {
    BarPoiChart,
    BarRoadChart,
    LineChart,
    LineChartCrowd,
    DashboardChart,
    DayChart,
    MonthChart,
    Banner,
    SplitNumber,
    Global,
    Panel
  }
};

</script>
<style>
div.fm-stretch {
  width: 100%;
  height: 100%;
}

div.flex-layout {
  display: flex;
  flex-flow: row nowrap;
  justify-content: space-between;
  align-items: center;
}

div.flex-layout-v {
  display: flex;
  flex-flow: column nowrap;
  justify-content: space-between;
  align-items: center;
}

div.float {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none;
}

div.flex-layout>div.col {
  width: 100%;
  height: 100%;
}

div.flex-layout-v>div.row {
  width: 100%;
  height: 100%;
}

div.legendContainer {
  display: flex;
  align-items: center;
}

div.legendContainer .img-btn {
  cursor: pointer;
  pointer-events: auto;
}

div.legendContainer div.legend {
  padding: 5px;
}

div.legendContainer div.legend>div {
  color: #fff;
  font-weight: 500;
  display: inline-block;
  line-height: 18px;
  cursor: pointer;
  pointer-events: auto;
  padding: 10px;
}

div.legendContainer div.legend span {
  display: inline-block;
  width: 16px;
  height: 16px;
  vertical-align: bottom;
  border-radius: 5px;
}

.num-text {
  margin-left: 11px;
  color: #fd8e20;
}

.center-content {
  width: 1013px;
  height: 264px;
  z-index: 9999;
  background: url(./assets/middle.png) no-repeat top/contain;
}

.center-content .summary {
  font-size: 24px;
  color: #fff;
  padding: 10px 30px 10px 50px;
}

.top-title {
  flex: 0 0 122px;
  width: 533px;
  display: flex;
  justify-content: center;
  flex-wrap: nowrap;
}

.top-title.left {
  background: url(./assets/left_up.png) no-repeat center;
}

.top-title.right {
  background: url(./assets/right_up.png) no-repeat center;
}

.top-title>.text {
  width: 250px;
  height: 70px;
  color: #fd8e2a;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 26px;
  font-weight: bold;
}

.top-title.left>.text {
  padding-left: 70px;
}

.top-title.right>.text {
  padding-right: 70px;
}

.bottom-title {
  flex: 0 0 92px;
  width: 624px;
  display: flex;
  align-items: flex-end;
}

.bottom-title.left {
  background: url(./assets/left_down.png) no-repeat center;
}

.bottom-title.right {
  justify-content: flex-end;
  background: url(./assets/right_down.png) no-repeat center;
}

.bottom-title>.text {
  color: #fd8e20;
  font-size: 24px;
  font-weight: bold;
  line-height: 46px;
  width: 566px;
  text-align: center;
}

div.inlineChart {
  display: inline-block;
  width: 48%;
}


/*覆盖cesium的默认样式，不显示下方工具条*/

.cesium-viewer-bottom {
  display: none;
}

</style>
