<template>
  <div class="chart-content">
    <div class="chart" :id="cahrtId"></div>
    <div class="dataDetail">
      <div><span>{{this.dashboard.titleName}}</span></div>
      <div><span>更新<span class="focusDetail">{{this.dashboard.dataUpdate}}</span>{{this.dashboard.dataUnit}}</span></div>
      <div><span>新增<span class="focusDetail" >{{this.dashboard.dataAdd}}</span>{{this.dashboard.dataUnit}}</span></div>
    </div>
  </div>
</template>
<script>
import echarts from 'echarts'

export default {
  name: 'test',
  props: ['dashboard'],
  data() {
    return {
      chart: null,
      cahrtId: 'dashboardId',
      titleName: '',
      dataUpdate: '更新 0',
      dataAdd: '新增 0'
    }
  },
  methods: {
    // 绘制表格
    drawGraph() {
      this.titleName = this.dashboard.titleName;
      this.dataUpdate = this.dashboard.dataUpdate
      this.dataAdd = this.dashboard.dataAdd
      if (!this.chart) {
        this.chart = echarts.init(document.getElementById(this.dashboard.cahrtId))
      }
      this.chart.showLoading();

      var addSpareData = function(data, color){
          var spareData = (Number(data[0]) + Number(data[1]))*0.35;
          console.log(spareData);
          return [{
            value: data[0], 
            name: '更新',
            itemStyle: {
              normal: {
                label : {
                  show : false
                },
                labelLine : {
                  show : false
                },
                color: color
              },
              emphasis: {
                label : {
                  show : false,
                  position: 'inner'
                },
                labelLine : {
                  show : false
                },
              }
            }
          },{
            value: data[1],
            name: '新增',
            itemStyle: {
              normal: {
                label : {
                  show : false
                },
                labelLine : {
                  show : false
                },
                color: 'rgba(237,237,237,0.5)'
              },
              emphasis: {
                label : {
                  show : false,
                  position: 'inner'
                },
                labelLine : {
                  show : false
                },
              }
            }
          },{
            value: spareData,
            name: '', 
            itemStyle: {
              normal: {
                label : {
                  show : false
                },
                labelLine : {
                  show : false
                },
                color: 'rgba(0,0,0,0)'
              }
            }
          }]
      }

      let seriesData = [{
          name: '',
          type: 'pie',
          startAngle: 224,
          radius : ['75%', '90%'],
          smooth: true,
          tooltip: {
            show: false
          },
          hoverAnimation: false,
          data: addSpareData([this.dashboard.dataUpdate, this.dashboard.dataAdd], this.dashboard.styleColor)
        }];

      this.chart.setOption({
        backgroundColor: 'rgba(128, 128, 128, 0)',
        title: {},
        legend: {},
        series: seriesData
      })
      this.chart.hideLoading()
    }
  },
  mounted() {},
  watch: {
    dashboard: {
      handler(curVal, oldVal) {
        this.cahrtId = this.dashboard.cahrtId;
        this.$nextTick(function() {
          var self = this;
          setTimeout(function(){
            self.drawGraph()
          },200)
        })　　　　
      },
      deep: true
    }
  }
}

</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.chart-content {
  color: #DDD;
  width: 200px;
  position: relative;
  display: inline-block;
}

.chart-content .chart {
  width: 200px;
  height: 170px;
}

.chart-content .dataDetail{
  position: absolute;
  left: 20%;
  top: 25%;
  font-size: 12px;
  text-align: center;
  width: 120px;
}
.chart-content .dataDetail .focusDetail{
  color: #FD8E20;
  font-size: 13px;
}
.chart-content .dataDetail div{
  /*margin-top: 4px */
}
</style>
