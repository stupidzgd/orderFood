<template>
  <div class="cartcontrol">
    <transition name="move">
      <div v-show="food.count>0" class="cart-decrease" @click.stop.prevent="decreaseCart">
        <span class="inner icon-remove_circle_outline"></span>
      </div>
    </transition>
    <div class="cart-count" v-show="food.count>0">{{food.count}}</div>
    <div class="cart-add icon-add_circle" @click.stop.prevent="addCart"></div>
  </div>
</template>

<script>
import Vue from 'vue';

export default {
  props: {
    food: {
      type: Object
    }
  },
  methods: {
    addCart(event) {
      if (!event._constructed) {
        return;
      }
      // food本身没有count，设置count属性
      if (!this.food.count) {
        Vue.set(this.food, 'count', 1);
      } else {
        this.food.count++;
      }
      // 把按钮的位置派发给父级，用于定位下落起点
      this.$emit('add', event.target);
    },
    decreaseCart(event) {
      if (!event._constructed) {
        return;
      }
      if (this.food.count) {
        this.food.count--;
      }
    }
  }
};
</script>

<style lang="less">
@import "../../common/less/index.less";

.cartcontrol {
  font-size: 0;
  .cart-decrease {
    display: inline-block;
    padding: 6px;
    opacity: 1;
    transform: translate3d(0, 0, 0) rotate(0deg);
    &.move-enter-active,
    &.move-leave-active {
      transition: 0.4s all linear;
    }
    &.move-enter,
    &.move-leave-to {
      transform: translate3d(24px, 0, 0) rotate(180deg);
    }
    .inner {
      display: inline-block;
      line-height: 24px;
      font-size: 24px;
      color: rgb(0, 160, 220);
    }
  }
  .cart-count {
    display: inline-block;
    vertical-align: top;
    padding-top: 6px;
    width: 12px;
    text-align: center;
    font-size: 10px;
    line-height: 24px;
    color: rgb(147, 153, 159);
  }
  .cart-add {
    display: inline-block;
    padding: 6px;
    line-height: 24px;
    font-size: 24px;
    color: rgb(0, 160, 220);
  }
}
</style>
