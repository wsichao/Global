<template>
  <div class="chart-content">
    <div id='myDayChart'>
    </div>
  </div>
</template>
<script>
import echarts from 'echarts'
// require('echarts/theme/dark');

export default {
  props: ['dayProduce'],
  data() {
    return {
      dataShadow: [] // 用于阴影柱状图显示
    }
  },
  data() {
    return {
      chart: null,
      seriesData: [],
      yAxisData: this.dayProduce.yAxis // ['新增POI100个','更新PIO3342个','新增道路23公里','更新道路342公里']
    }
  },
  methods: {
    // 获取背景柱状图数组
    shadowMax() {
      var max = 0
      var maxArr = []
      for (let i = 0; i < this.dayProduce.barData.length; i++) {
        if (max < this.dayProduce.barData[i]) {
          max = this.dayProduce.barData[i]
        }
      }
      max = Math.ceil(max)
      this.dayProduce.barData.forEach(function(item, index, arr) {
        maxArr[index] = max
      })
      return maxArr
    },
    // 绘制表格
    drawGraph() {
      this.dataShadow = this.shadowMax()

      if (!this.chart) {
        this.chart = echarts.init(document.getElementById('myDayChart'))
      }

      this.chart.showLoading()
      this.chart.setOption({
        backgroundColor: 'rgba(128, 128, 128, 0)',
        grid: {
          left: 80,
          right: 100,
          top: 8,
          bottom: 26
        },
        legend: {
          itemWidth: 14,
          bottom: 0,
          data: [
            {
              name: '日均线',
              icon: 'roundRect',
              textStyle: { color: '#FFFFFF' }
            }
          ]
        },
        xAxis: {
          show: false
        },
        yAxis: [
          {
            data: this.dayProduce.yAxis,
            boundaryGap: true, // 坐标轴两边留空白
            axisLine: {
              show: true
            },
            axisTick: {
              show: false
            },
            axisLabel: {
              fontSize: 14,
              fontWeight: 'bold',
              color: '#DDD'
            }
          },
          {
            data: this.dayProduce.unit,
            boundaryGap: true, // 坐标轴两边留空白
            offset: -30,
            axisLine: {
              show: false
            },
            axisTick: {
              show: true
            },
            axisLabel: {
              fontSize: 14,
              fontWeight: 'bold',
              color: '#FD8E20'
            }
          }
        ],
        series: [
          {
            // For shadow
            type: 'bar',
            itemStyle: {
              normal: {
                color: 'rgba(255,255,255,0.15)',
                barBorderRadius: [0, 5, 5, 0]
              }
            },
            barGap: '-100%', // 两个柱子重叠
            barCategoryGap: '80%', // 柱子之间的间距
            data: this.dataShadow,
            animation: false
          },
          {
            type: 'bar',

            itemStyle: {
              normal: {
                color: '#39f',
                barBorderRadius: [0, 5, 5, 0]
              }
            },
            data: this.dayProduce.barData // [100, 90, 80, 70]
          },
          {
            name: '日均线',
            type: 'line',
            symbol: 'none', // 去掉点
            smooth: true,
            itemStyle: {
              normal: {
                color: '#fd8e2a'
              }
            },
            data: this.dayProduce.lineData
          }
        ]
      })
      this.chart.hideLoading()
    }
  },
  mounted() {},
  watch: {
    dayProduce: {
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
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.chart-content {
  color: #ddd;
  width: 400px;
  position: relative;
  display: inline-block;
}

#myDayChart {
  width: 400px;
  height: 200px;
}
</style>
