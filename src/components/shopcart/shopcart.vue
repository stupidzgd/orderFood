<template>
  <div class="shopcart">
    <div class="content">
      <div class="content-left">
        <div class="logo-wrapper">
          <div class="logo">
            <i class="icon-shopping_cart"></i>
          </div>
        </div>
        <div class="price">￥0</div>
        <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
      </div>
      <div class="content-right">
        <div class="pay" :class="payClass">￥{{minPrice}}元起送</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    }
  },
  computed: {
    payClass() {
      return 'not-enough';
    }
  }
};
</script>

<style lang="less">
.shopcart {
  position: fixed;
  left: 0;
  bottom: 0;
  z-index: 50;
  height: 48px;
  width: 100%;
  .content {
    display: flex;
    height: 100%;
    background: #141d27;
    font-size: 0;
    color: rgba(255, 255, 255, 0.4);
    .content-left {
      flex: 1;
      .logo-wrapper {
        position: relative;
        display: inline-block;
        top: -10px;
        width: 56px;
        height: 56px;
        padding: 6px;
        margin: 0 12px;
        box-sizing: border-box;
        border-radius: 50%;
        background: #141d27;
        .logo {
          width: 100%;
          height: 100%;
          border-radius: 50%;
          background-color: #2b343c;
          text-align: center;
          &.highlight {
            background: rgb(0, 160, 220);
          }
          .icon-shopping_cart {
            font-size: 24px;
            line-height: 44px;
            &.highlight {
              color: #fff;
            }
          }
        }
      }
      .price {
        display: inline-block;
        margin-top: 12px;
        padding-right: 12px;
        vertical-align: top;
        box-sizing: border-box;
        line-height: 24px;
        font-size: 16px;
        font-weight: 700;
        border-right: 1px solid rgba(255, 255, 255, 0.1);
      }
      .desc {
        display: inline-block;
        margin: 12px 0 0 12px;
        vertical-align: top;
        font-size: 10px;
        line-height: 24px;
      }
    }
    .content-right {
      flex: 0 0 105px;
      width: 105px;
      .pay {
        height: 48px;
        line-height: 48px;
        font-size: 12px;
        font-weight: 700;
        text-align: center;
        &.not-enough {
          background: #2b333b;
        }
        &.enough {
          background: #00b43c;
          color: #fff;
        }
      }
    }
  }
}
</style>
