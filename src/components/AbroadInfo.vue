<template>
  <div class="info-box" :style="{top: top, left: left}">
    <div class="desc">
      <div class="desc-p">
      <p class="desc-title">{{ info.summary.devName }}</p>
      <p>开发时间：{{ info.summary.devDate }}</p>
      <p>范围：{{ info.summary.scope }}</p>
      <p>道路总里程：{{ info.summary.roadSum }}</p>
      <p>POI数量：{{ info.summary.poiSum }}</p>
      </div>
    </div>
    <div class="img">
      <img :src="imgSrc">
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
      return `./static/images/abroad/map_${this.info.id}.png`;
    },
    top() {
      return `${this.anchor[1] - 200}px`;
    },
    left() {
      return `${this.anchor[0] + 40}px`;
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
  display: flex;
  flex-flow: column nowrap;
  position: absolute;
  height: 424px;
  width: 392px;
  padding-top: 40px;
  padding-bottom: 30px;
  background-image: url(../../static/images/abroad/frame.png);
  box-shadow: 0 0 50px -5px #2fa9fc;
}

.info-box > .desc {
  padding: 0px 60px 10px 60px;
  color: #2fa9fc;
}

.desc-p {
  padding: 10% 0;
  align-items: center;
  justify-content: center;
}

.desc-p>p {
  margin: 10px 0px;
}

.desc-title {
  font-size: 24px;
}

.info-box > .img {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 8px;
}

.img > img {
  max-width: 100%;
  max-height: 100%;
}

</style>
