<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList">
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
          <div class="pay" :class="payClass" @click.stop.prevent="pay">{{payDesc}}</div>
        </div>
      </div>
      <div class="ball-container">
        <div v-for="(ball, index) in balls" :key="index">
          <transition name="drop" @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop">
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
      <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <p class="title">购物车</p>
            <span class="empty" @click="empty">清空</span>
          </div>
          <div class="list-content" ref="listContent">
            <ul>
              <transition-group name="delete-list">
              <li class="food" v-for="(food, index) in selectFoods" :key="index">
                <span class="name">{{food.name}}</span>
                <div class="price">
                  <span>￥{{food.price * food.count}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food" @add="addFood"></cartcontrol>
                </div>
              </li>
              </transition-group>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <transition name="fade">
      <div class="list-mask" @click="hideList" v-show="listShow"></div>
    </transition>
  </div>
</template>

<script>
import cartcontrol from '@/components/cartcontrol/cartcontrol';
import BScroll from 'better-scroll';

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
      fold: true,
      balls: [
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        },
        {
          show: false
        }
      ],
      dropBalls: []
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
      if (show) {
        if (!this.scroll) {
          this._initScroll();
        } else {
          this.scroll.refresh();
        }
      }
      return show;
    }
  },
  methods: {
    addFood(target) {
      this.drop(target);
    },
    // 触发小球下落动画
    drop(target) {
      for (let i = 0; i < this.balls.length; i++) {
        let ball = this.balls[i];
        if (!ball.show) {
          ball.show = true;
          // 把下落的起始点挂到小球属性上
          ball.el = target;
          this.dropBalls.push(ball);
          return;
        }
      }
    },
    beforeDrop(el) {
      let count = this.balls.length;
      // 遍历小球，初始化小球位置到event.target，就是点击添加食品的位置
      while (count--) {
        let ball = this.balls[count];
        if (ball.show) {
          let rect = ball.el.getBoundingClientRect();
          let x = rect.left - 32;
          let y = -(window.innerHeight - rect.top - 22);
          el.style.display = '';
          el.style.webkitTransform = `translate3d(0,${y}px,0)`;
          el.style.transform = `translate3d(0,${y}px,0)`;
          let inner = el.getElementsByClassName('inner-hook')[0];
          inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
          inner.style.transform = `translate3d(${x}px,0,0)`;
        }
      }
    },
    dropping(el, done) {
      // 手动触发重排,确保小球被正常渲染到初始位置
      /* eslint-disable no-unused-vars */
      let reflow = el.offsetHeight;
      this.$nextTick(() => {
        el.style.webkitTransform = 'translate3d(0,0,0)';
        el.style.transform = 'translate3d(0,0,0)';
        let inner = el.getElementsByClassName('inner-hook')[0];
        inner.style.webkitTransform = 'translate3d(0,0,0)';
        inner.style.transform = 'translate3d(0,0,0)';
        el.addEventListener('transitionend', done);
      });
    },
    afterDrop(el) {
      let ball = this.dropBalls.shift();
      if (ball) {
        ball.show = false;
        el.style.display = 'none';
      }
    },
    toggleList() {
      if (!this.totalCount) {
        return;
      }
      this.fold = !this.fold;
    },
    hideList() {
      this.fold = true;
    },
    empty() {
      this.selectFoods.forEach((food) => {
        food.count = 0;
      });
    },
    pay() {
      if (this.totalPrice < this.minPrice) {
        return;
      }
      alert(`支付${this.totalPrice}元`);
    },
    _initScroll() {
      this.scroll = new BScroll(this.$refs.listContent, {
        click: true
      });
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
    &.fold-enter-active,
    &.fold-leave-active {
      transition: 0.4s all linear;
    }
    &.fold-enter,
    &.fold-leave-to {
      transform: translate3d(0, 0, 0);
    }
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
      background-color: #fff;
      .food {
        position: relative;
        padding: 12px 0;
        box-sizing: border-box;
        height: 48px;
        &.delete-list-enter-active,
        &.delete-list-leave-active {
          transition: height 0.1s ease-in-out;
        }
        &.delete-list-enter,
        &.delete-list-leave-to {
          height: 0;
        }
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
  .ball-container {
    .ball {
      position: fixed;
      left: 32px;
      bottom: 22px;
      z-index: 200;
      // transition: all 0.4s  cubic-bezier(.41,.09,.69,-0.28);
      transition: all 0.4s  cubic-bezier(.72,-0.29,.77,-0.1);
      .inner {
        width: 16px;
        height: 16px;
        border-radius: 50%;
        background: rgb(0, 160, 220);
        transition: all 0.4s linear;
      }
    }
  }
}
.list-mask {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 40;
  opacity: 1;
  background-color: rgba(7, 17, 27, 0.6);
  &.fade-enter-active,
  &.fade-leave-active {
    transition: all 0.5s;
  }
  &.fade-enter,
  &.fade-leave-to {
    opacity: 0;
  }
}
</style>
