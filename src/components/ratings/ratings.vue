<template>
    <div class="ratings" ref="ratings">
      <div class="ratings-content">
        <div class="overview">
          <div class="overview-left">
            <div class="score">{{seller.score}}</div>
            <div class="title">综合评分</div>
            <div class="text">高于周边商家{{seller.rankRate}}%</div>
          </div>
          <div class="overview-right">
            <div class="score-wrapper">
              <span class="title">服务态度</span>
              <star :score="seller.serviceScore" :size="36"></star>
              <span class="score">{{seller.serviceScore}}</span>
            </div>
            <div class="score-wrapper">
              <span class="title">商品评分</span>
              <star :score="seller.foodScore" :size="36"></star>
              <span class="score">{{seller.foodScore}}</span>
            </div>
            <div class="delivery-wrapper">
              <span class="title">送达时间</span><span class="time">{{seller.deliveryTime}}分钟</span>
            </div>
          </div>
        </div>
        <split></split>
        <ratingselect @toggle="toggleContent" @select="selectRatingType"
        :ratings="ratings" :onlyContent="onlyContent" :desc="desc" :selectType="selectType"></ratingselect>
        <div class="ratingselect-wrapper">
          <ul>
            <li class="rating-item" v-for="(rating, index) in ratings" v-show="needShow(rating.rateType, rating.text)" :key="index">
              <div class="avatar-wrapper">
                <img :src="rating.avatar" class="avatar" width="28" height="28">
              </div>
              <div class="content">
                <h1 class="name">{{rating.username}}</h1>
                <div class="star-wrapper">
                  <star :size="24" :score="rating.score"></star>
                  <span class="delivery" v-show="rating.deliveryTime">{{rating.deliveryTime}}分钟送达</span>
                </div>
                <div class="text" v-show="rating.text">{{rating.text}}</div>
                <div class="recommend" v-show="rating.recommend.length > 0">
                  <i :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></i>
                  <span class="item" v-for="(item, index) in rating.recommend" :key="index">{{item}}</span>
                </div>
                <div class="time">{{rating.rateTime | formatDate}}</div>
              </div>
            </li>
          </ul>
        </div>
      </div>
    </div>
</template>

<script>
import split from '@/components/split/split';
import star from '@/components/star/star';
import ratingselect from '@/components/ratingselect/ratingselect';
import BScroll from 'better-scroll';
import axios from 'axios';
import { formatDate } from '@/common/js/date.js';

let ALL = 2;
let ERR_OK = 0;
const debug = process.env.NODE_ENV !== 'production';

export default {
  props: {
    seller: Object
  },
  data() {
    return {
      selectType: ALL,
      onlyContent: true,
      desc: {
        all: '全部',
        positive: '满意',
        negative: '不满意'
      },
      ratings: []
    };
  },
  components: {
    split,
    star,
    ratingselect
  },
  created() {
    const url = debug ? '/api/goods' : 'http://stupidzgd.com/sell/api/ratings';
    // const url = debug ? '/api/ratings' : 'http://127.0.0.1:9000/api/ratings';
    axios.get(url).then((response) => {
      response = response.data;
      if (response.errno === ERR_OK) {
        this.ratings = response.data;
        this.$nextTick(() => {
          this.scroll = new BScroll(this.$refs.ratings, {
            click: true
          });
        });
      }
    });
  },
  methods: {
    needShow(type, text) {
      if (this.onlyContent && !text) {
        return false;
      }
      if (this.selectType === ALL) {
        return true;
      } else {
        return this.selectType === type;
      }
    },
    toggleContent() {
      this.onlyContent = !this.onlyContent;
      this.$nextTick(() => {
        this.scroll.refresh();
      });
    },
    selectRatingType(type) {
      this.selectType = type;
      this.$nextTick(() => {
        this.scroll.refresh();
      });
    }
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

.ratings {
  position: absolute;
  top: 174px;
  left: 0;
  bottom: 0;
  width: 100%;
  overflow: hidden;
  .overview {
    display: flex;
    padding: 18px 0;
    .overview-left {
      flex: 0 0 137px;
      padding: 6px 0;
      border-right: 1px solid rgba(7, 17, 27, 0.1);
      text-align: center;
      @media screen and (max-width: 320px) {
        flex: 0 0 120px;
        width: 120px;
      }
      .score {
        line-height: 28px;
        font-size: 24px;
        color: rgb(255, 153, 0);
        margin-bottom: 6px;
        font-weight: 700;
      }
      .title {
        font-size: 12px;
        line-height: 12px;
        color: rgb(7, 17, 27);
        margin-bottom: 8px;
      }
      .text {
        font-size: 10px;
        color: rgb(147, 153, 159);
        line-height: 10px;
        margin-bottom: 6px;
      }
    }
    .overview-right {
      flex: 1;
      padding: 6px 0 6px 24px;
      @media only screen and (max-width: 320px) {
        padding-left: 6px;
      }
      .score-wrapper {
        margin-bottom: 8px;
        font-size: 0;
        .title {
          margin-right: 12px;
          vertical-align: top;
          font-size: 12px;
          color: rgb(7, 17, 27);
          line-height: 18px;
        }
        .star {
          display: inline-block;
          margin-right: 12px;
        }
        .score {
          font-size: 12px;
          vertical-align: top;
          color: rgb(255, 153, 0);
          line-height: 18px;
        }
      }
      .delivery-wrapper {
        .title {
          margin-right: 12px;
          font-size: 12px;
          color: rgb(7, 17, 27);
          line-height: 18px;
        }
        .time {
          font-size: 12px;
          line-height: 18px;
          color: rgb(147, 153, 159);
        }
      }
    }
  }
  .ratingselect-wrapper {
    padding: 0 18px;
    .rating-item {
      display: flex;
      padding: 18px 0;
      .border-1px(rgba(7, 17, 27, 0.1));
      .avatar-wrapper {
        flex: 0 0 28px;
        width: 28px;
        margin-right: 12px;
        img {
          border-radius: 50%;
        }
      }
      .content {
        position: relative;
        flex: 1;
        .name {
          margin-bottom: 4px;
          font-size: 10px;
          color: rgb(7, 17, 27);
          line-height: 12px;
        }
        .star-wrapper {
          margin-bottom: 6px;
          font-size: 0;
          .star {
            display: inline-block;
          }
          .delivery {
            margin-left: 6px;
            font-size: 10px;
            color: rgb(147, 153, 159);
            line-height: 12px;
          }
        }
        .text {
          margin-bottom: 8px;
          font-size: 12px;
          color: rgb(7, 17, 27);
          line-height: 18px;
        }
        .recommend {
          line-height: 16px;
          font-size: 0;
          .icon-thumb_up,
          .item {
            display: inline-block;
            margin: 0 8px 4px 0;
            font-size: 9px;
          }
          .icon-thumb_up {
            color: rgb(0, 160, 220);
          }
          .item {
            padding: 0 6px;
            border: 1px solid rgba(7, 17, 27, 0.1);
            border-radius: 1px;
            color: rgb(147, 153, 159);
            background: #fff;
          }
        }
        .time {
          position: absolute;
          right: 0;
          top: 0;
          font-size: 10px;
          color: rgb(147, 153, 159);
          line-height: 12px;
        }
      }
    }
  }
}
</style>
