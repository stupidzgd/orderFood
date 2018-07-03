<template>
  <div class="shopcart">
    <div class="content">
      <div class="content-left">
        <div class="logo-wrapper">
          <div class="logo" :class="{'highlight': totalCount > 0}">
            <i class="icon-shopping_cart" :class="{'highlight': totalCount > 0}"></i>
          </div>
          <div class="num" v-show="totalCount > 0">{{totalCount}}</div>
        </div>
        <div class="price" :class="{'highlight': totalPrice > 0}">￥{{totalPrice}}</div>
        <div class="desc">另需配送费￥{{deliveryPrice}}元</div>
      </div>
      <div class="content-right">
        <div class="pay" :class="payClass">{{payDesc}}</div>
      </div>
    </div>
    <transition name="fold">
    <div class="shopcart-list" v-show="listShow">
      <div class="list-header">
        <p class="title">购物车</p>
        <span class="empty">清空</span>
      </div>
      <div class="list-content">
        <ul>
          <li class="food" v-for="(food, index) in selectFoods" :key="index">
            <span class="name">{{food.name}}</span>
            <div class="price">
              <span>￥{{food.price * food.count}}</span>
            </div>
            <div class="cartcontrol-wrapper">
              <cartcontrol :food="food"></cartcontrol>
            </div>
          </li>
        </ul>
      </div>
    </div>
    </transition>
  </div>
</template>

<script>
import cartcontrol from '@/components/cartcontrol/cartcontrol';

export default {
  props: {
    selectFoods: {
      type: Array,
      default() {
        return [
          {
            price: 10,
            count: 1
          }
        ];
      }
    },
    deliveryPrice: {
      type: Number,
      default: 0
    },
    minPrice: {
      type: Number,
      default: 0
    }
  },
  data() {
    return {
      fold: true
    };
  },
  computed: {
    payClass() {
      if (this.totalPrice >= this.minPrice) {
        return 'enough';
      } else {
        return 'not-enough';
      }
    },
    payDesc() {
      if (this.totalPrice === 0) {
        return `￥${this.minPrice}元起送`;
      } else if (this.totalPrice >= this.minPrice) {
        return '去结算';
      } else {
        return `还差￥${this.minPrice - this.totalPrice}元起送`;
      }
    },
    totalPrice() {
      let total = 0;
      this.selectFoods.forEach((food) => {
        total += food.price * food.count;
      });
      return total;
    },
    totalCount() {
      let count = 0;
      this.selectFoods.forEach((food) => {
        count += food.count;
      });
      return count;
    },
    listShow() {
      if (!this.totalCount) {
        this.hideList();
        return false;
      }
      let show = !this.fold;
      return show;
    }
  },
  methods: {
    toggleList() {
      if (!this.totalCount) {
        return;
      }
      this.fold = !this.fold;
    },
    hideList() {
      this.fold = false;
    }
  },
  components: {
    cartcontrol
  }
};
</script>

<style lang="less">
@import "../../common/less/index.less";

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
        .num {
          position: absolute;
          top: 0;
          right: 0;
          width: 24px;
          height: 16px;
          line-height: 16px;
          text-align: center;
          border-radius: 16px;
          color: #fff;
          font-weight: 700;
          font-size: 9px;
          background: #f01414;
          box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4);
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
        &.highlight {
          color: #fff;
        }
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
  .shopcart-list {
    position: absolute;
    left: 0;
    top: 0;
    z-index: -1;
    width: 100%;
    background-color: #f3f5f7;
    transform: translate3d(0, -100%, 0);
    .list-header {
      height: 40px;
      line-height: 40px;
      padding: 0 18px;
      border-bottom: 1px solid rgba(7, 17, 27, 0.1);
      .title {
        float: left;
        line-height: 40px;
        font-size: 14px;
        color: rgb(7, 17, 27);
      }
      .empty {
        float: right;
        line-height: 40px;
        font-size: 12px;
        color: rgb(0, 160, 220);
      }
    }
    .list-content {
      padding: 0 18px;
      max-height: 217px;
      overflow: hidden;
      .food {
        position: relative;
        padding: 12px 0;
        box-sizing: border-box;
        .border-1px(rgba(7, 17, 27, 0.1));
        .name {
          font-size: 14px;
          color: rgb(7, 17, 27);
          line-height: 24px;
        }
        .price {
          position: absolute;
          right: 90px;
          bottom: 12px;
          line-height: 24px;
          font-size: 14px;
          color: rgb(240, 20, 20);
        }
        .cartcontrol-wrapper {
          position: absolute;
          right: 0;
          bottom: 6px;
        }
      }
    }
  }
}
</style>
