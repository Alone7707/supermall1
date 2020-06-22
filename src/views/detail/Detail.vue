<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"/>
    <scroll class="content" ref="scroll" @scroll="contentScroll" :probe-type="3">
      <detail-swiper :top-images="topImages"/>
      <detail-base-info :goods="goods"/>
      <detail-comment-info :comment-info="commentInfo" ref="comment"/>
      <detail-shop-info :shop="shop"/>
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"/>
      <detail-param-info :param-info="paramInfo" ref="params"/>
      <goods-list :goods="recommends" ref="recommend"/>
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
    <detail-bottom-bar @addCart="addToCart" />
  </div>
</template>

<script>
  import DetailNavBar from "./childComps/DetailNavBar";
  import DetailSwiper from "./childComps/DetailSwiper";
  import DetailBaseInfo from "./childComps/DetailBaseInfo";
  import DetailShopInfo from "./childComps/DetailShopInfo";
  import DetailGoodsInfo from "./childComps/DetailGoodsInfo";
  import DetailParamInfo from "./childComps/DetailParamInfo";
  import DetailCommentInfo from "./childComps/DetailCommentInfo";
  import DetailBottomBar from "./childComps/DetailBottomBar";

  import Scroll from "components/common/scroll/Scroll";
  import GoodsList from "components/content/goods/GoodsList";
  import BackTop from "components/content/backTop/BackTop";

  import {getDetail, Goods, Shop, GoodsParam} from "network/detail";
  import {getRecommend} from "network/detail";
  import {debounce} from "common/utils";
  import {itemListenerMixin} from "common/mixin"

  export default {
    name: "Detail",
    data() {
      return {
        iid: null,
        topImages: [],
        goods: {},
        shop: {},
        detailInfo: {},
        paramInfo: {},
        commentInfo: {},
        recommends: [],
        themeTopYs: [],
        getThemeTopY: null,
        cruuentIndex: 0,
        isShowBackTop: false,
      }
    },
    mixins: [itemListenerMixin],
    components: {
      DetailNavBar,
      DetailSwiper,
      DetailBaseInfo,
      DetailShopInfo,
      DetailGoodsInfo,
      DetailParamInfo,
      DetailCommentInfo,
      DetailBottomBar,
      Scroll,
      GoodsList,
      BackTop
    },
    created() {
      // 1.保存传入的iid
      const iid= this.$route.params.iid
      this.iid = iid

      // 2.根据iid请求详情数据
      getDetail(iid).then(res => {

        // 2.1.获取数据
        const data = res.result

        // 2.2.去除轮播图的数据
        this.topImages = data.itemInfo.topImages

        // 2.3.获取商品信息
        this.goods = new Goods(data.itemInfo, data.columns, data.shopInfo.services)

        // 2.4.创建店铺信息的对象
        this.shop = new Shop(data.shopInfo)

        // 2.5.保存商品的详情数据
        this.detailInfo = data.detailInfo

        // 2.6.获取商品的参数信息
        this.paramInfo = new GoodsParam(data.itemParams.info, data.itemParams.rule)

        // 2.7.取出评论的信息
        if (data.rate.cRate !== 0) {
          this.commentInfo = data.rate.list[0]
        }

        // this.$nextTick(() => {
        //   // 根据最新的数据，对应的DOM是已经被渲染出来
        //   // 但是图片依然是没有加载完(目前获取到oddsetTop不包含其中的图片)
        //   // offsetTio值不对的时候，都是因为图片的问题
        //   this.themeTopYs = []
        //   this.themeTopYs.push(0)
        //   this.themeTopYs.push(this.$refs.params.$el.offsetTop)
        //   this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
        //   this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
        //   console.log(this.themeTopYs);
        // })
      })

      // 3.请求推荐数据
      getRecommend().then(res => {
        this.recommends = res.data.list
      })

      // 4.给getThemeTopY赋值
      this.getThemeTopY = debounce(() => {
        this.themeTopYs = []
        this.themeTopYs.push(0)
        this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
        this.themeTopYs.push(this.$refs.params.$el.offsetTop)
        this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
        this.themeTopYs.push(Number.MAX_VALUE)
        // console.log(this.themeTopYs);
      }, 500)
    },
    mounted() {
      const newRefresh = debounce(this.$refs.scroll.refresh, 500)

      this.itemImgListener = () => {
        newRefresh()
      }

    },
    updated() {
      this.detailImageLoad()
    },
    destroyed() {
      this.$bus.$on('itemImgLoad', this.itemImgListener)
    },
    methods: {

      imageLoad() {
        // this.$refs.scroll.refresh()
        this.newRefresh()
      },
      contentScroll(position) {
        // 1.获取y值
        const positionY = -position.y
        // 2.positionY和主题中值进行对比
        let length = this.themeTopYs.length
        for (let i = 0; i < length-1; i++) {
          if (this.currentIndex !== i && (positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1])) {
            this.currentIndex = i
            this.$refs.nav.currentIndex = this.currentIndex
          }

          // if (this.currentIndex !== i && ((i < length -1 && positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1]) || (i === length -1 && positionY >= this.themeTopYs[i]))) {
          //   this.currentIndex = i
          //   this.$refs.nav.currentIndex = this.currentIndex
          // }
        }
        this.isShowBackTop = (-position.y) > 1000
      },
      detailImageLoad() {
        this.newRefresh()
        this.getThemeTopY()
      },
      titleClick(index) {
        this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 300)
      },
      backClick() {
        this.$refs.scroll.scrollTo(0, 0)
      },
      addToCart() {
        // 1.获取购物车需要展示得信息
        const product = {}
        product.image = this.topImages[0]
        product.title = this.goods.title
        product.desc = this.goods.desc
        product.price = this.goods.realPrice
        product.iid = this.iid

        // 2.讲商品添加到购物车里
        // this.$store.cartList.push(product)
        // this.$store.commit('addCart', product)
        this.$store.dispatch('addCart', product)
      }
    }
  }
</script>

<style scoped>
  #detail {
    height: 100vh;
    position: relative;
    z-index: 10;
    background-color: #fff;
  }
  .content {
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
    overflow: hidden;
  }
  .detail-nav {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 20;
    background-color: #fff;
  }
</style>
