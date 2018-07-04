<template>
  <transition name="move">
    <div v-show="showFlag" class="food" ref="food">
      <div class="food-content">
        <div class="image-header">
          <img :src="food.image">
          <div class="back" @click="hide">
            <i class="icon-arrow_lift"></i>
          </div>
        </div>
        <div class="content">
          <h1 class="title">{{food.name}}</h1>
          <div class="detail">
            <span class="sell-count">月售{{food.sellCount}}份</span>
            <span class="rating">好评率{{food.rating}}%</span>
          </div>
          <div class="price">
            <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
          </div>
          <div class="cartcontrol-wrapper">
            <cartcontrol :food="food" @add="addFood"></cartcontrol>
          </div>
          <div class="buy" v-show="food.count===0 || !food.count" @click="addFirst">加入购物车</div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import cartcontrol from '@/components/cartcontrol/cartcontrol';
import BScroll from 'better-scroll';
import Vue from 'vue';

export default {
  props: {
    food: {
      type: Object
    }
  },
  data() {
    return {
      showFlag: false
    };
  },
  methods: {
    show() {
      this.showFlag = true;
      this.$nextTick(() => {
        if (!this.scroll) {
          this.scroll = new BScroll(this.$refs.food, {
            click: true
          });
        } else {
          this.scroll.refresh();
        }
      });
    },
    hide() {
      this.showFlag = false;
    },
    addFood(target) {
      this.$emit('add', target);
    },
    addFirst(event) {
      if (!event._constructed) {
        return;
      }
      this.$emit('add', event.target);
      Vue.set(this.food, 'count', 1);
    }
  },
  components: {
    cartcontrol
  }
};
</script>

<style lang="less">
.food {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 48px;
  z-index: 30px;
  width: 100%;
  background-color: #fff;
  &.move-enter-active,
  &.move-leave-active {
    transition: 0.4s all;
  }
  &.move-enter,
  &.move-leave-to {
    transform: translate3d(100%, 0, 0);
  }
  .image-header {
    position: relative;
    width: 100%;
    height: 0;
    padding-top: 100%;
    img {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }
    .back {
      position: absolute;
      left: 0;
      top: 10px;
      .icon-arrow_lift {
        display: block;
        font-size: 20px;
        padding: 10px;
        color: #fff;
      }
    }
  }
  .content {
    position: relative;
    padding: 18px;
    .title {
      font-size: 14px;
      font-weight: 700;
      color: rgb(7, 17, 27);
      line-height: 14px;
    }
    .detail {
      margin-top: 8px;
      font-size: 0;
      .sell-count,
      .rating {
        margin-right: 12px;
        font-size: 10px;
        line-height: 10px;
        color: rgb(147, 153, 159);
      }
    }
    .price {
      margin-top: 18px;
      font-weight: 700;
      line-height: 24px;
      .now {
        margin-right: 12px;
        font-size: 14px;
        color: rgb(240, 20, 20);
        line-height: 24px;
      }
      .old {
        font-size: 10px;
        line-height: 24px;
        color: rgb(147, 153, 159);
        text-decoration: line-through;
      }
    }
    .cartcontrol-wrapper {
      position: absolute;
      right: 12px;
      bottom: 12px;
    }
    .buy {
      position: absolute;
      right: 18px;
      bottom: 18px;
      font-size: 10px;
      height: 12px;
      line-height: 12px;
      padding: 6px 12px;
      color: #fff;
      background-color: rgb(0, 160, 220);
      border-radius: 12px;
    }
  }
}
</style>
