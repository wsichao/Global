<template>
  <div class="info-box" :style="{top: top, left: left, background-image: url(imgSrc)}">
    <div class="text">
      <p>{{ description }}</p>
    </div>
  </div>
</template>
<script>
export default {
  name: 'InfoBox',
  props: ['position': Array, 'type': String],
  data: function() {
    return {

    };
  },
  computed: {
    imgSrc() {
      return `../static/images/${this.type}_introduction.png`
    },
    top() {
      return '100px';
    },
    left() {
      return '100px';
    }
  },
  watch: {

  },
  methods: {

  },
  mounted() {

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
  background: url(../static/images/laowo_introduction.png);
}

</style>
