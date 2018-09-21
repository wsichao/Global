<template>
  <div class="info-box" :style="{top: top, left: left}">
    <div class="content">
      <div class="desc">
        <p>开发时间：{{ info.summary.devDate }}</p>
        <p>范围：{{ info.summary.scope }}</p>
        <p>道路总里程：{{ info.summary.roadSum }}</p>
        <p>POI数量：{{ info.summary.poiSum }}</p>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'AbroadInfo',
  props: {
    info: {
      type: Object,
      required: true
    },
    anchor: {
      type: Array,
      required: true
    }
  },
  computed: {
    imgSrc() {
      return `./static/images/abroad/${this.info.id}_introduction.png`;
    },
    top() {
      return `${this.anchor[1] - 200}px`;
    },
    left() {
      return `${this.anchor[0]}px`;
    }
  },
  mounted() {
    console.log(this.info);
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
  background-image: url(../../static/images/abroad/frame.png);
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
