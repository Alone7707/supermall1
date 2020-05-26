<template>
  <div class="recommend">
    <div v-for="item in recommends" class="recommends-item">
      <a :href="item.link">
        <img :src="item.image" alt="">
        <div>{{item.title}}</div>
      </a>
    </div>
  </div>
</template>

<script>
  export default {
    name: "RecommendView",
    props: {
      recommends: {
        type: Array,
        default() {
          return []
        }
      }
    }
  }
</script>

<style scoped>
  .recommend {
    display: flex;
    padding: 10px 0 20px;
    text-align: center;
    border-bottom: 7px solid #eee;
  }
  .recommends-item {
    flex: 1;
  }

  .recommend img {
    width: 80%;
    margin-bottom: 5px;
  }
</style>
