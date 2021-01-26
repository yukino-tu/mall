<template>
  <div class="cart-list">
    <scroll class="content" ref="scroll">
      <cart-list-item v-for="(item,index) in cartList" :key="index"
      :item-info="item" />
    </scroll>
  </div>
</template>

<script>
  import CartListItem from 'views/cart/childComps/CartListItem'

  import Scroll from 'components/common/scroll/Scroll'

  import {mapGetters} from 'vuex'

  export default {
    name: "CartList",
    components: {
      Scroll,
      CartListItem
    },
    computed: {
      ...mapGetters(['cartList'])
    },
    activated() {
      this.$refs.scroll.refresh()
    }
  }
</script>

<style scoped>
  .cart-list{
    position: relative;
    /* height: 100%; */
  }
  .content{
    height: calc(100vh - 44px - 49px);
    overflow-y: hidden;
  }
</style>