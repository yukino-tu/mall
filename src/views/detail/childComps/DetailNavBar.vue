<template>
  <div>
    <nav-bar class="detail-nav">
      <div slot="left" class="back" @click="backClick">
        <img src="~assets/img/common/back.svg" alt="">
      </div>
      <div slot="center" class="title">
        <div v-for="(item,index) in titles" :key="index"
        class="title-item"
        :class="{active: index === currentIndex}"
        @click="titleClick(index)">{{item}}</div>
      </div>
  </nav-bar>
  </div>
</template>

<script>
  import NavBar from 'components/common/navbar/NavBar'

  export default {
    name: 'DetailNavBar',
    components: {
      NavBar,
    },
    data() {
      return {
        titles: ['商品', '参数', '评论', '搜索'],
        currentIndex: 0,
      }
    },
    methods: {
      backClick() {
        this.$router.back()
      },
      titleClick(index) {
        this.currentIndex = index
        this.$emit('titleClick',index)
      },
    },
  }
</script>

<style scoped>
  .detail-nav{
    position: relative;
    z-index: 9;
    background-color: #fff;
  }

  .back img{
    margin-top: 10px;
  }
  .title {
    display: flex;
    font-size: 12px;
  }
  .title-item{
      flex: 1;
  }
  .active{
    color: var(--color-high-text);
  }
</style>