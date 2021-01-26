<template>
  <div id="bottom-bar">
    <div class="check-content">
      <check-button :is-checked="isCheckAll" @click.native="checkClick" class="check-btn" />
      <span>全选</span>
    </div>
    <div class="price price-cancel">
      合计: {{totalPrice}}
    </div>
    <div class="calculate" @click="calculateClick">
      去计算: {{checkLength}}
    </div>
  </div>
</template>

<script>
  import CheckButton from 'components/content/checkButton/CheckButton'

  import {mapGetters} from 'vuex'

  export default {
    name: "CartBottomBar",
    components: {
      CheckButton,
      
    },
    computed: {
      ...mapGetters(['cartList']),
      // 计算总价
      totalPrice() {
        return '￥' + this.cartList.filter(item => {
          return item.checked
        }).reduce((preValue, item) => {
          return preValue + item.realPrice * item.count
        },0).toFixed(2)
      },
      // 计算选中个数
      checkLength() {
        return this.cartList.filter(item => item.checked).length
      },
      isCheckAll() {
        if(this.cartList.length === 0) {
          return false
        }
        // return !this.cartList.find(item => item.checked)
        return this.cartList.every(item => item.checked)
      }
    },
    methods: {
      checkClick() {
        if(this.isCheckAll) { 
          this.cartList.forEach(item => item.checked = false)
        }else {
          this.cartList.forEach(item => item.checked = true)
        }
      },
      calculateClick() {
        if(!this.isCheckAll){
          this.$toast.show('请选择默认商品',1500)
        }
      }
    }
  }
</script>

<style scoped>
   #bottom-bar {
    height: 40px;
    position: fixed;
    display: flex;
    bottom: 49px;
    left: 0;
    right: 0;
    /*background-color: #eee;*/
    box-shadow: 0px 1px 5px rgba(0, 0, 0, 0.3);
  }

  #bottom-bar .check-content {
    width:auto;
    display: flex;
    flex: 1;
    line-height:40px;
    justify-content: center;
    align-items: center;
  }

  #bottom-bar .check-content .check-btn {
    /*width: 20px;*/
    /*height: 20px;*/
    /*margin-right: 5px;*/
    line-height: 20px;
    display: flex;
    /* flex: 1; */
    /*position: relative;*/
    /*justify-content: center;*/
    align-items: center;
  }

  .price {
    text-align: center;
    flex: 2;
    /*height: 40px;*/
    line-height: 40px;
  }

  .price span {
    color: var(--color-tint);
  }

  .calculate {
    height: 40px;
    line-height: 40px;
    background-color: var(--color-high-text);
    color: #fff;
    text-align: center;
    flex: 1;
  }
  .price-cancel {
    color: white;
    background-color: var(--color-tint);
  }


  
</style>