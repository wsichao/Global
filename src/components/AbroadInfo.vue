<template>
  <div class="info-box" :style="{top: top, left: left, 'background-image': imgSrc}">
    <div class="content">
      <div class="desc">
        <p>开发时间：{{ desc.date }}</p>
        <p>范围：{{ desc.scope }}</p>
        <p>道路总里程：{{ desc.roadSum }}</p>
        <p>POI数量：{{ desc.poiSum }}</p>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'AbroadInfo',
  props: {
    data: {
      type: Object,
      required: true
    },
    anchor: {
      type: Array,
      required: true
    }
  },
  data() {
    return {};
  },
  computed: {
    imgSrc() {
      return `url(./static/images/abroad/${this.data.id}_introduction.png)`;
    },
    top() {
      return `${this.anchor[1] - 200}px`;
    },
    left() {
      return `${this.anchor[0]}px`;
    },
    desc() {
      if (this.type === 'laowo') {
        return {
          date: '2015年',
          scope: '全境，包括17个省、直辖市',
          roadSum: '4.5万KM，其中高等级道路里程1.5万KM',
          poiSum: '2.3万， 重点POI全面覆盖'
        };
      } else if (this.type === 'jianpuzhai') {
        return {
          date: '2016年',
          scope: '全境， 包括12个重点城市及其国家主干道',
          roadSum: '8万KM',
          poiSum: '3万，重点POI全面覆盖'
        };
      }
      else if (this.type === 'miandian') {
        return {
          date: '2018年',
          scope: '全境',
          roadSum: '18.6万KM',
          poiSum: '规划重点POI全面覆盖'
        };
      }
      else if (this.type === 'yuenan') {
        return {
          date: '2018年',
          scope: '全境',
          roadSum: '16万KM',
          poiSum: '规划重点POI全面覆盖'
        };
      }
      else if (this.type === 'taiwan') {
        return {
          date: '2018年',
          scope: '全境',
          roadSum: '13万KM',
          poiSum: '规划重点POI全面覆盖'
        };
      }

      return {};
    }
  }
};

</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.info-box {
  display: inline;
  position: absolute;
  height: 488px;
  width: 386px;
  padding: 50px;
  pointer-events: none;
}

.info-box .content {
  width: 100%;
  height: 100%;
  pointer-events: auto;
}

.info-box .desc {
  padding: 60px 40px;
  color: #2fa9fc;
}

</style>
