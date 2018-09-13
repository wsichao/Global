<template>
  <div class="info-box" :style="{top: top, left: left, 'background-image': imgSrc}">
    <div class="content">
      <p>开发时间：{{ desc.date }}</p>
      <p>范围：{{ desc.scope }}</p>
      <p>道路总里程：{{ desc.roadSum }}</p>
      <p>POI数量：{{ desc.poiSum }}</p>
    </div>
  </div>
</template>
<script>
export default {
  name: 'InfoBox',
  props: {
    'type': {
      type: String,
      required: true
    }
  },
  data() {
    return {};
  },
  computed: {
    imgSrc() {
      return `url(../static/images/${this.type}_introduction.png)`;
    },
    top() {
      return '300px';
    },
    left() {
      return '1000px';
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
  height: 588px;
  width: 486px;
}

.info-box .content {
    padding: 120px 100px;
    color: #2fa9fc;
}

</style>
