<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center" >购物街</div>
    </nav-bar>
    <tab-control class="tab-control" :titles="['流行','新款','精选']" @tabClick="tabClick" ref="topTabControl" v-show="isFixTable" />
    <scroll class="content" ref="scroll" :probe-type="3" @scroll="contentScroll" :pull-up-load="true" @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad" />
      <recommend-view :recommends="recommends"/>
      <feature-view />
      <tab-control :titles="['流行','新款','精选']" @tabClick="tabClick" ref="tabControl" />
      <goods-list :goods="showGoods" />
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop" />
  </div>
</template>

<script>
import NavBar from 'components/common/navbar/NavBar'
import TabControl from 'components/content/tabControl/TabControl'

import { getHomeMultidata, getHomeGoods } from 'network/home'
import {debounce} from 'common/utils'
import { itemListener,backTopMinxin } from 'common/mixin'

import HomeSwiper from 'views/home/childComps/HomeSwiper'
import RecommendView from 'views/home/childComps/RecommendView'
import FeatureView from 'views/home/childComps/FeatureView'
import GoodsList from 'components/content/goods/GoodsList'
import Scroll from 'components/common/scroll/Scroll'

export default {
  name: 'Home',
  components: {
    NavBar,
    HomeSwiper,
    RecommendView,
    FeatureView,
    TabControl,
    GoodsList,
    Scroll,
  },
  mixins: [itemListener, backTopMinxin],
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] },
      },
      currentType: 'pop',
      tableOffsetTop: 0,
      isFixTable: false,
      saveY: 0
    }
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list
    },
  },
  created() {
    // 1.请求多个数据
    this.msGetHomeMultidata()
    // 2.请求商品数据
    this.msGetHomeGoods('pop')
    this.msGetHomeGoods('new')
    this.msGetHomeGoods('sell')
  },
  activated() {
    this.$refs.scroll.refresh()
    this.$refs.scroll.scrollTo(0,this.saveY,0)
  },
  deactivated() {
    // 获取scroll.y的值
    this.saveY = this.$refs.scroll.getScrollY()

    // 离开时关闭事件总线的监听
    this.$bus.$off('itemImgLoad',this.itemImgListener)
  },
  mounted() {
    // const refresh = debounce(this.$refs.scroll.refresh,50)
    // this.$bus.$on("itemImageLoad",()=>{
    //   refresh()
    // })
  },
  methods: {
    //事件监听相关方法    
    tabClick(index) {
      switch (index) {
        case 0:
          this.currentType = 'pop'
          break
        case 1:
          this.currentType = 'new'
          break
        case 2:
          this.currentType = 'sell'
      }
      // 让两个tabcontrol的currentindex保持一致
      this.$refs.topTabControl.currentIndex = index
      this.$refs.tabControl.currentIndex =index
    },
    contentScroll(position){
      // 判断backTop是否显示
      this.isShowBackTop = (-position.y) > 1000
      // 判断tableControl是否吸顶
      this.isFixTable = (-position.y) > this.tableOffsetTop
    },
    loadMore(){
      this.msGetHomeGoods(this.currentType)
    },
    swiperImageLoad(){
      this.tableOffsetTop = this.$refs.tabControl.$el.offsetTop
    },
    //网络请求相关的方法
    msGetHomeMultidata() {
      getHomeMultidata().then((res) => {
        this.banners = res.data.banner.list
        this.recommends = res.data.recommend.list
      })
    },
    msGetHomeGoods(type) {
      const page = this.goods[type].page + 1
      getHomeGoods(type, page).then((res) => {
        // console.log(res);
        this.goods[type].list.push(...res.data.list)
        this.goods[type].page += 1

      // 完成上拉加载更多
        this.$refs.scroll.finishPullUp()
      })
    },
    
  },
}
</script>

<style scoped>
  #home{
    /* padding-top: 44px; */
    height: 100vh;
    position: relative;
  }

  .home-nav {
  background-color: var(--color-tint);
  color: #fff;

  /* position: fixed;
  left: 0;
  right: 0;
  top: 0;
  z-index: 9; */
}

.tab-control {
  position: relative;
  z-index: 9;
  background-color: #fff;
}

.content{
  overflow: hidden;

  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
  right: 0;
}
</style>
