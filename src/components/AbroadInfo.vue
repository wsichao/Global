<template>
  <div class="info-box" :style="{top: top, left: left, 'background-image': imgSrc}">
    <div class="content">
      <div class="desc">
        <p>开发时间：{{ data.summary.devDate }}</p>
        <p>范围：{{ data.summary.scope }}</p>
        <p>道路总里程：{{ data.summary.roadSum }}</p>
        <p>POI数量：{{ data.summary.poiSum }}</p>
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
