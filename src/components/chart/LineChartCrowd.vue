<template>
  <div class="chart-content">
    <div id='crowdChart' style="padding-top: 8px;padding-bottom: 8px;'">
    </div>
  </div>
</template>
<script>
import echarts from 'echarts'
// require('echarts/theme/dark');

export default {
  name: 'test',
  props: ['chartData'],
  data() {
    return {
      chart: null
    }
  },
  methods: {
    // 绘制表格
    drawGraph() {
      if (!this.chart) {
        this.chart = echarts.init(document.getElementById('crowdChart'))
      }
      this.chart.showLoading()
      let legendData = [
        { name: `采集道路${this.chartData.crowdRoadLen}公里`, icon: 'roundRect' },
        { name: `采集poi${this.chartData.crowdPoiNum}个`, icon: 'roundRect' }
      ];

      /**
       * 获取渐变色
       */
      function getLinerStyle(satrtColor, endColor){
        return new echarts.graphic.LinearGradient(
          0, 0, 0, 1,
          [
            {offset: 0.2, color: endColor},
            {offset: 0.9, color: satrtColor}
          ]
        )
      }

      let seriesData = [{
          name: `采集道路${this.chartData.crowdRoadLen}公里`,
          type: 'line',
          symbol: 'none',
          // symbolSize: 6,
          smooth: true,
          itemStyle: {
            normal: {
              color: getLinerStyle('#d6e6e6', '#FFC82C'),
              areaStyle: {
                type: 'default'
              }
            }
          },
          data: this.chartData.lineData[0]
        },
        {
          name: `采集poi${this.chartData.crowdPoiNum}个`,
          type: 'line',
          symbol: 'none',
          // symbolSize: 6,
          smooth: true,
          itemStyle: {
            normal: {
              color: getLinerStyle('#d6e6e6', '#39f'),
              areaStyle: {
                type: 'default'
              }
            }
          },
          data: this.chartData.lineData[1]
        }
      ]

      this.chart.setOption({
        backgroundColor: 'rgba(128, 128, 128, 0)',
        grid: {
          left: 15,
          right: 20,
          top: 45,
          bottom: 24
        },
        title: {},
        legend: {
          orient: 'horizontal',
          x: 'left',
          right: 20,
          top: 0,
          itemWidth: 14,
          data: legendData,
          textStyle: { color: '#DDD' }
        },
        xAxis: {
          data: this.chartData.xAxis,
          boundaryGap: false, // 坐标轴两边不留空白
          axisLine: {
            show: false
          },
          axisTick: {
            show: false
          },
          axisLabel: {
            color: '#DDDDDD',
            interval: 0 // 强制显示
          }
        },
        yAxis: {
          show: false,
          boundaryGap: false // 坐标轴两边不留空白
        },/* [{
          show: false,
          axisLabel : {
            textStyle: {
              color: '#FFF'
            }
          },
          axisLine: {
            lineStyle: {
              color: '#4D4F56'
            }
          },
          splitLine: {
            show: false
          }
        }], */
        series: seriesData
      })
      this.chart.hideLoading()
    }
  },
  mounted() {},
  watch: {
    chartData: {
      handler(curVal, oldVal) {
        this.$nextTick(function() {
          this.drawGraph()
        })　　　　
      },
      　　　　deep: true
    }
  }
}

</script>
<style scoped>
.chart-content {
  color: #DDD;
  width: 400px;
  display: inline-block;
}

#crowdChart {
  width: 400px;
  height: 140px;
}

</style>
