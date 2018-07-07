<template>
    <div class="seller" ref="seller">
      <div class="seller-content">
        <div class="overview">
          <h1 class="title">{{seller.name}}</h1>
          <div class="desc">
            <star :score="seller.score" :size="36"></star>
            <span class="rating-count">({{seller.ratingCount}})</span>
            <span class="sell-count">月售{{seller.sellCount}}单</span>
          </div>
          <ul class="delivery">
            <li class="item">
              <h2 class="title">起送价</h2>
              <div class="content">{{seller.minPrice}}<span class="min">元</span></div>
            </li>
            <li class="item">
              <h2 class="title">商家配送</h2>
              <div class="content">{{seller.deliveryPrice}}<span class="min">元</span></div>
            </li>
            <li class="item">
              <h2 class="title">平均配送时间</h2>
              <div class="content">{{seller.deliveryTime}}<span class="min">分钟</span></div>
            </li>
          </ul>
          <div class="favorite" @click="toggleFavorite">
            <span class="icon-favorite" :class="{'active': favorite}"></span>
            <span class="text">{{favoriteText}}</span>
          </div>
        </div>
        <split></split>
        <div class="bulletin">
          <div class="title">公告与活动</div>
          <div class="content-wrapper">
            <p class="content">{{seller.bulletin}}</p>
          </div>
          <ul v-if="seller.supports" class="supports">
            <li v-for="(support, index) in seller.supports" :key="index" class="support-item">
              <i class="icon" :class="classMap[support.type]"></i><span class="desc">{{support.description}}</span>
            </li>
          </ul>
        </div>
        <split></split>
        <div class="pics">
          <div class="title">商家实景</div>
          <div class="pic-wrapper" ref="picWrapper">
            <ul v-if="seller.supports" class="pic-list" ref="picList">
              <li v-for="(pic, index) in seller.pics" class="pic-item" :key="index">
                <img :src="pic" width="120" height="90">
              </li>
            </ul>
          </div>
        </div>
        <split></split>
        <div class="seller-message">
          <div class="title">商家信息</div>
          <ul v-if="seller.infos" class="info-wrapper">
            <li v-for="(info, index) in seller.infos" class="info-item" :key="index">
              {{info}}
            </li>
          </ul>
        </div>
      </div>
    </div>
</template>

<script>
import BScroll from 'better-scroll';
import star from '@/components/star/star';
import split from '@/components/split/split';
import { saveToLocal, loadFromLocal } from '@/common/js/store';

export default {
  props: {
    seller: Object
  },
  data() {
    return {
      favorite: (() => {
        return loadFromLocal(this.seller.id, 'favorite', false);
      })()
    };
  },
  created() {
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
  },
  mounted() {
    this.$nextTick(() => {
      this._initScroll();
      this._initPicScroll();
    });
  },
  methods: {
    toggleFavorite(event) {
      if (!event._constructed) {
        return;
      }
      this.favorite = !this.favorite;
      saveToLocal(this.seller.id, 'favorite', this.favorite);
    },
    _initScroll() {
      if (!this.scroll) {
        this.scroll = new BScroll(this.$refs.seller, {
          click: true
        });
      } else {
        this.scroll.refresh();
      }
    },
    _initPicScroll() {
      if (this.seller.pics) {
        let picWidth = 120;
        let margin = 6;
        let width = (picWidth + margin) * this.seller.pics.length - margin;
        this.$refs.picList.style.width = width + 'px';
        this.$nextTick(() => {
          if (!this.picScroll) {
            this.picScroll = new BScroll(this.$refs.picWrapper, {
              scrollX: true,
              eventPassthrough: 'vertical'
            });
          } else {
            this.picScroll = new BScroll(this.$refs.picWrapper, {
              scrollX: true,
              eventPassthrough: 'vertical'
            });
          }
        });
      }
    }
  },
  computed: {
    favoriteText() {
      return this.favorite ? '已收藏' : '收藏';
    }
  },
  watch: {
    seller() {
      this.$nextTick(() => {
        this._initScroll();
        this._initPicScroll();
      });
    }
  },
  components: {
    star,
    split
  }
};
</script>

<style lang="less">
@import "../../common/less/index.less";

.seller {
  position: absolute;
  left: 0;
  top: 174px;
  bottom: 0;
  width: 100%;
  overflow: hidden;
  .overview {
    position: relative;
    padding: 18px;
    .title {
      font-size: 14px;
      line-height: 14px;
      color: rgb(7, 17, 27);
    }
    .desc {
      padding: 8px 0 18px 0;
      .border-1px(rgba(7, 17, 27, 0.1));
      font-size: 0;
      .star {
        display: inline-block;
        margin-right: 8px;
      }
      .rating-count {
        margin-right: 12px;
        vertical-align: top;
        font-size: 10px;
        color: rgb(77, 85, 93);
        line-height: 18px;
      }
      .sell-count {
        vertical-align: top;
        font-size: 10px;
        color: rgb(77, 85, 93);
        line-height: 18px;
      }
    }
    .delivery {
      display: flex;
      padding: 18px 0 0 0;
      .item {
        flex: 1;
        text-align: center;
        border-right: 1px solid rgba(7, 17, 27, 0.1);
        &:last-child {
          border-right: none;
        }
        .title {
          margin-bottom: 4px;
          font-size: 10px;
          color: rgb(147, 153, 159);
          line-height: 10px;
        }
        .content {
          font-size: 24px;
          color: rgb(7, 17, 27);
          line-height: 24px;
          .min {
            font-size: 10px;
          }
        }
      }
    }
    .favorite {
      position: absolute;
      width: 50px;
      right: 11px;
      top: 18px;
      text-align: center;
      .icon-favorite {
        display: block;
        margin-bottom: 4px;
        font-size: 24px;
        line-height: 24px;
        color: #d4d6d9;
        &.active {
          color: rgb(240, 20, 20);
        }
      }
      .text {
        display: block;
        font-size: 10px;
        color: rgb(77, 85, 93);
        line-height: 10px;
      }
    }
  }
  .bulletin {
    padding: 18px 18px 0 18px;
    .title {
      margin-bottom: 8px;
      font-size: 14px;
      line-height: 14px;
      color: rgb(7, 17, 27);
    }
    .content-wrapper {
      padding: 0 12px 16px 12px;
      .border-1px(rgba(7, 17, 27, 0.1));
      .content {
        font-size: 12px;
        color: rgb(240, 20, 20);
        line-height: 24px;
      }
    }
    .supports {
      .support-item {
        padding: 16px 12px;
        font-size: 0;
        .border-1px(rgba(7, 17, 27, 0.1));
        &:last-child {
          .border-none();
        }
        .icon {
          display: inline-block;
          margin-right: 6px;
          width: 16px;
          height: 16px;
          background-size: 16px 16px;
          background-repeat: no-repeat;
          &.decrease {
            .bg-image("decrease_4");
          }
          &.discount {
            .bg-image("discount_4");
          }
          &.special {
            .bg-image("special_4");
          }
          &.invoice {
            .bg-image("invoice_4");
          }
          &.guarantee {
            .bg-image("guarantee_4");
          }
        }
        .desc {
          vertical-align: top;
          font-size: 12px;
          color: rgb(7, 17, 27);
          line-height: 16px;
        }
      }
    }
  }
  .pics {
    padding: 18px;
    overflow: hidden;
    .title {
      margin-bottom: 12px;
      font-size: 14px;
      line-height: 14px;
      color: rgb(7, 17, 27);
    }
    .pic-wrapper {
      width: 100%;
      overflow: hidden;
      white-space: nowrap;
      .pic-list {
        font-size: 0;
        .pic-item {
          display: inline-block;
          margin-right: 6px;
        }
      }
    }
  }
  .seller-message {
    padding: 18px 18px 0 18px;
    .title {
      padding-bottom: 12px;
      font-size: 14px;
      line-height: 14px;
      color: rgb(7, 17, 27);
      .border-1px(rgba(7, 17, 27, 0.1));
    }
    .info-item {
      padding: 16px 12px;
      font-size: 12px;
      color: rgb(7, 17, 27);
      line-height: 16px;
      .border-1px(rgba(7, 17, 27, 0.1));
      &:last-child {
        .border-none();
      }
    }
  }
}
</style>
