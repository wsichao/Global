<template>
  <div class="chart-content">
    <div id="myLineChart" style="padding: 8px 0px;">
    </div>
  </div>
</template>
<script>
import echarts from "echarts";
// require('echarts/theme/dark');

export default {
  name: "test",
  props: ["chartData"],
  data() {
    return {
      chart: null
      // xAxisData: ['1月','2月','3月','4月','5月','6月','7月','8月','9月','10月','11月','12月']
    };
  },
  methods: {
    // 绘制表格
    drawGraph() {
      if (!this.chart) {
        this.chart = echarts.init(document.getElementById("myLineChart"));
      }
      this.chart.showLoading();
      let legendData = [
        { name: `轨迹13508.063亿个`, icon: "roundRect" },
        { name: `情报${this.chartData.inforTotal}个`, icon: "roundRect" },
        { name: `反馈${this.chartData.userTotal}个`, icon: "roundRect" },
        { name: `互联网${this.chartData.webTotal}个`, icon: "roundRect" }
      ];
      let seriesData = [
        {
          name: `轨迹13508.063亿个`,
          type: "line",
          symbol: "none",
          smooth: true,
          itemStyle: {
            normal: {
              areaStyle: {
                type: "default",
                color: getLinerStyle("rgba(0,0,0,0.1)", "#FF9933")
              },
              color: "#FF9933"
            }
          },
          data: [
            1738.82293882,
            1426.64095209,
            1403.30852207,
            813.82310678,
            1366.00084594,
            1244.18313622,
            1132.15832078,
            1911.12552555,
            1878.86507383,
            593.13944326,
            0,
            0
          ]
        },
        {
          name: `情报${this.chartData.inforTotal}个`,
          type: "line",
          symbol: "none", // 去掉点
          smooth: true,
          itemStyle: {
            normal: {
              areaStyle: {
                type: "default",
                color: getLinerStyle("rgba(0,0,0,0.1)", "#FDE22A")
              },
              color: "#FDE22A"
            }
          },
          data: this.chartData.lineData[0] // [120, 132, 101, 134, 90, 230, 210, 120, 132, 101, 134, 90]
        },
        {
          name: `反馈${this.chartData.userTotal}个`,
          type: "line",
          symbol: "none",
          smooth: true,
          itemStyle: {
            normal: {
              areaStyle: {
                type: "default",
                color: getLinerStyle("rgba(0,0,0,0.1)", "#3493FF")
              },
              color: "#3493FF"
            }
          },
          data: this.chartData.lineData[1] // [120, 182, 191, 234, 290, 330, 310, 220, 182, 191, 234, 290]
        },
        {
          name: `互联网${this.chartData.webTotal}个`,
          type: "line",
          symbol: "none",
          smooth: true,
          itemStyle: {
            normal: {
              areaStyle: {
                type: "default",
                color: getLinerStyle("rgba(0,0,0,0.1)", "#01B31E")
              },
              color: "#01B31E"
            }
          },
          data: this.chartData.lineData[2] //[120, 232, 201, 154, 190, 330, 410, 150, 232, 201, 154, 190]
        }
      ];

      /**
       * 获取渐变色
       */
      function getLinerStyle(satrtColor, endColor) {
        return new echarts.graphic.LinearGradient(0, 0, 0, 1, [
          { offset: 0.2, color: endColor },
          { offset: 0.9, color: satrtColor }
        ]);
      }

      this.chart.setOption({
        backgroundColor: "rgba(128, 128, 128, 0)",
        grid: {
          left: 14,
          right: 15,
          // top: 20,
          bottom: 30
        },
        title: {
          // text: '累积值',
          // left:20,
          // top: 20,
          // textStyle: {
          //   color: '#FFFFFF'
          // }
        },
        legend: {
          orient: "horizontal",
          x: "left",
          itemWidth: 14,
          data: legendData,
          textStyle: { color: "#DDD" }
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
            color: "#DDDDDD",
            interval: 0 // 强制显示
          }
        },
        yAxis: {
          show: false
        },
        series: seriesData
      });
      this.chart.hideLoading();
    }
  },
  mounted() {},
  watch: {
    chartData: {
      handler(curVal, oldVal) {
        this.$nextTick(function() {
          this.drawGraph();
        });
      },
      deep: true
    }
  }
};
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.chart-content {
  color: #ddd;
  width: 400px;
  display: inline-block;
}

#myLineChart {
  width: 400px;
  height: 140px;
}
</style>
