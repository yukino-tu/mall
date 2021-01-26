<template>
  <div id="detail">
    <detail-nav-bar @titleClick="titleClick" ref="nav" />
    <scroll class="content" :probe-type="3" @scroll="contentScroll" ref="scroll">
      <detail-swiper :top-images="topImages" />
      <detail-base-info :goods="goods" />
      <detail-shop-info :shop="shopInfo" />
      <detail-goods-info :detail-info="detailInfo" @detailImageLoad="detailImageLoad" />
      <detail-param-info :param-info="paramInfo" ref="params" />
      <detail-comment-info :comment-info="commentInfo" ref="comment" />
      <goods-list :goods="recommends" ref="recommend" />
    </scroll>
    <detail-bottom-bar @addCart="detailAddCart" />
    <back-top @click.native="backClick" v-show="isShowBackTop" />
  </div>
</template>

<script>
import DetailNavBar from 'views/detail/childComps/DetailNavBar'
import DetailSwiper from 'views/detail/childComps/DetailSwiper'
import DetailBaseInfo from 'views/detail/childComps/DetailBaseInfo'
import DetailShopInfo from 'views/detail/childComps/DetailShopInfo'
import DetailGoodsInfo from 'views/detail/childComps/DetailGoodsInfo'
import DetailParamInfo from 'views/detail/childComps/DetailParamInfo'
import DetailCommentInfo from 'views/detail/childComps/DetailCommentInfo'
import DetailBottomBar from 'views/detail/childComps/DetailBottomBar'

import {
  getDetail,
  getRecommend,
  Goods,
  Shop,
  GoodsParam,
} from 'network/detail'

import { itemListener,backTopMinxin } from 'common/mixin'
import { debounce } from 'common/utils'
import {mapActions} from 'vuex'

import Scroll from 'components/common/scroll/Scroll'
import GoodsList from 'components/content/goods/GoodsList'

export default {
  name: 'Detail',
  components: {
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailGoodsInfo,
    DetailShopInfo,
    Scroll,
    DetailGoodsInfo,
    DetailParamInfo,
    DetailCommentInfo,
    GoodsList,
    DetailBottomBar
  },
  mixins: [itemListener, backTopMinxin],
  data() {
    return {
      iid: null,
      topImages: [],
      goods: {},
      shopInfo: {},
      detailInfo: {},
      paramInfo: {},
      commentInfo: {},
      recommends: [],
      themeTopYs: [],
      getThemeTopYs: null,
      currentIndex: 0,
    }
  },
  created() {
    //1、保存传入的iid
    this.iid = this.$route.params.id
    //2、根据iid请求详情数据
    getDetail(this.iid).then((res) => {
      // console.log(res);
      //1、获取顶部图片 的 轮播数据
      const data = res.result
      this.topImages = data.itemInfo.topImages

      //2、获取商品信息
      this.goods = new Goods(
        data.itemInfo,
        data.columns,
        data.shopInfo.services
      )

      //3、创建店铺信息
      this.shopInfo = new Shop(data.shopInfo)

      //4、获取商品 详细信息
      this.detailInfo = data.detailInfo

      //5、保存参数信息
      this.paramInfo = new GoodsParam(
        data.itemParams.info,
        data.itemParams.rule
      )

      // 6.取出评论信息
      if (data.rate.CRate !== 0) {
        this.commentInfo = data.rate.list[0]
      }
    })
    //3.请求推荐数据
    getRecommend().then((res) => {
      this.recommends = res.data.list
    })

    // 给getThemeTopYs赋值
    this.getThemeTopYs = debounce(() => {
      ;(this.themeTopYs = []), this.themeTopYs.push(0)
      this.themeTopYs.push(this.$refs.params.$el.offsetTop)
      this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
      this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
      this.themeTopYs.push(Number.MAX_VALUE)
    }, 500)
  },
  destroyed() {
    this.$bus.$off('itemImgLoad', this.itemImgListener)
  },
  methods: {
    ...mapActions(['addCart']),
    detailImageLoad() {
      this.refresh()
      this.getThemeTopYs()
    },
    titleClick(index) {
      this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 200)
    },
    contentScroll(position) {
      const positionY = -position.y
      let length = this.themeTopYs.length
      for (let i = 0; i < length; i++) {
        // if (
        //   this.currentIndex !== i &&
        //   ((i < length - 1 &&
        //     positionY >= this.themeTopYs[i] &&
        //     positionY < this.themeTopYs[i + 1]) ||
        //     (i === length - 1 && positionY >= this.themeTopYs[i]))
        // ) {
        //   this.currentIndex = i
        //   this.$refs.nav.currentIndex = this.currentIndex
        // }
          if(this.currentIndex !== i &&(positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i+1])){
            this.currentIndex = i
            this.$refs.nav.currentIndex = this.currentIndex
          }
      }

      // 是否显示回到顶部
      this.isShowBackTop = (-position.y) > 1000
    },
    detailAddCart() {
      const product = {}
      product.image = this.topImages[0]
      product.title = this.goods.title
      product.desc = this.goods.desc
      product.price = this.goods.newPrice
      product.iid = this.iid;
      product.realPrice = this.goods.realPrice;
      console.log(product);
      // 将商品添加到购物车里面
      // this.$store.commit('addCart',product)
      // this.$store.dispatch('addCart',product)
      this.addCart(product).then(res => {
        console.log(res)
        this.$toast.show(res,2000)
      })

    }
  },
}
</script>

<style scoped>
#detail {
  height: 100vh;
  position: relative;
  z-index: 1;
  background-color: #fff;
}

.content {
  background-color: #fff;
  height: calc(100% - 44px - 49px);
}
</style>