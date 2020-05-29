<template>
  <div id="app">
    <router-view class="bottom"></router-view>
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
