<template>
  <div id="app">
    <keep-alive exclude="Detail">
      <router-view class="bottom"/>
    </keep-alive>
    <main-tab-bar/>
  </div>
</template>

<script>
import MainTabBar from "./components/content/mainTabbar/MainTabBar";

export default {
  name: 'App',
  components: {
    MainTabBar
  }
}
</script>

<style>
  @import "assets/css/base.css";
  .bottom {
    padding-bottom: 49px;
  }
</style>
