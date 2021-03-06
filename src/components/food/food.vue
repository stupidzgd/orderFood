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
          <transition name="fade">
            <div class="buy" v-show="food.count===0 || !food.count" @click="addFirst">加入购物车</div>
          </transition>
        </div>
        <split></split>
        <div class="info" v-show="food.info">
          <h1 class="title">商品信息</h1>
          <p class="text">{{food.info}}</p>
        </div>
        <split></split>
        <div class="rating">
          <h1 class="title">商品评价</h1>
          <ratingselect @toggle="toggleConent" @select="selectRating"
          :desc="desc" :onlyContent="onlyContent" :selectType="selectType" :ratings="food.ratings"></ratingselect>
          <div class="rating-wrapper">
            <ul v-show="food.ratings && food.ratings.length">
              <li v-for="(rating, index) in food.ratings" :key="index" v-show="needShow(rating.rateType, rating.text)" class="rating-item">
                <div class="user">
                  <span class="name">{{rating.username}}</span>
                  <img :src="rating.avatar" width="12" height="12" class="avatar">
                </div>
                <div class="time">{{rating.rateTime | formatDate}}</div>
                <p class="text">
                  <span :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></span>{{rating.text}}
                </p>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import cartcontrol from '@/components/cartcontrol/cartcontrol';
import BScroll from 'better-scroll';
import Vue from 'vue';
import Split from '@/components/split/split';
import ratingselect from '@/components/ratingselect/ratingselect';
import { formatDate } from '@/common/js/date.js';

const ALL = 2;

export default {
  props: {
    food: {
      type: Object
    }
  },
  data() {
    return {
      showFlag: false,
      // 下面三个属性都是用于评论切换组件的参数
      selectType: ALL,
      onlyContent: true,
      desc: {
        all: '全部',
        positive: '推荐',
        negative: '吐槽'
      }
    };
  },
  methods: {
    // 暴露给goods组件调用show
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
    },
    toggleConent() {
      this.onlyContent = !this.onlyContent;
      this.$nextTick(() => {
        this.scroll.refresh();
      });
    },
    selectRating(type) {
      this.selectType = type;
      this.$nextTick(() => {
        this.scroll.refresh();
      });
    },
    // 判断该条评论是否显示
    needShow(type, text) {
      if (this.onlyContent && !text) {
        return false;
      }
      if (this.selectType === ALL) {
        return true;
      } else {
        return this.selectType === type;
      }
    }
  },
  components: {
    cartcontrol,
    Split,
    ratingselect
  },
  filters: {
    formatDate(time) {
      let date = new Date(time);
      return formatDate(date, 'yyyy-MM-dd hh:mm');
    }
  }
};
</script>

<style lang="less">
@import "../../common/less/index.less";

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
        vertical-align: top;
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
      opacity: 1;
      &.fade-enter-active,
      &.fade-leave-active {
        transition: all 0.3s;
      }
      &.fade-enter,
      &.fade-leave-to {
        opacity: 0;
      }
    }
  }
  .info {
    padding: 18px;
    .title {
      font-size: 14px;
      color: rgb(7, 17, 27);
      line-height: 14px;
    }
    .text {
      margin: 6px 8px 0 8px;
      font-size: 12px;
      font-weight: 200;
      color: rgb(77, 85, 93);
      line-height: 24px;
    }
  }
  .rating {
    padding-top: 18px;
    .title {
      margin-left: 18px;
      font-size: 14px;
      color: rgb(7, 17, 27);
      line-height: 14px;
    }
    .rating-wrapper {
      padding: 0 18px;
      .rating-item {
        position: relative;
        padding: 16px 0;
        .border-1px(rgba(7, 17, 27, 0.1));
        .user {
          position: absolute;
          top: 16px;
          right: 0;
          line-height: 12px;
          font-size: 0;
          .avatar {
            border-radius: 50%;
          }
          .name {
            vertical-align: top;
            margin-right: 6px;
            font-size: 10px;
            color: rgb(147, 153, 159);
          }
        }
        .time {
          margin-bottom: 6px;
          font-size: 10px;
          color: rgb(147, 153, 159);
          line-height: 12px;
        }
        .text {
          line-height: 16px;
          font-size: 12px;
          color: rgb(7, 17, 27);
          .icon-thumb_up,
          .icon-thumb_down {
            margin-right: 4px;
            line-height: 16px;
            font-size: 12px;
          }
          .icon-thumb_up {
            color: rgb(0, 160, 220);
          }
          .icon-thumb_down {
            color: rgb(147, 153, 159);
          }
        }
      }
    }
  }
}
</style>
