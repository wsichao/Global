<template>
  <div class="fm-panel">
    <div class="header">
      <div class="title">{{title}}</div>
    </div>
    <div class="content">
      <div class="subTitle" v-show="subTitle">{{subTitle}}</div>
      <slot name="content"></slot>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'panel',
    props: ['title', 'subTitle'],
    data() {
      return {};
    },
  };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .fm-panel {
    display: flex;
    flex-flow: column nowrap;
    pointer-events: auto;
    padding-left: 26px;
  }
  .fm-panel>.header {
    display: flex;
    align-items: center;
  }
  .fm-panel>.header>.title {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 370px;
    height: 45px;
    padding: 0;
    margin-left: 38px;
    color: #FD8E2A;
    font-size: 18px;
    font-weight: bold;
    background: url(../assets/tab.png) no-repeat;
    background-size: 100% 100%;
  }
  .fm-panel>.content>.subTitle {
    display: flex;
    align-items: center;
    justify-content: center;
    color: #fff;
    font-size: 20px;
    margin-left: -220px;
  }
  .fm-panel>.content {
    flex: 1 0 100px;
    padding-left: 20px;
  }
</style>
