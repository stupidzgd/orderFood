<template>
  <div class="ratingselect">
    <div class="rating-type">
      <span @click="select(2, $event)" class="block positive" :class="{'active': selectType===2}">{{desc.all}}<span class="count">{{ratings.length}}</span></span>
      <span @click="select(0, $event)" class="block positive" :class="{'active': selectType===0}">{{desc.positive}}<span class="count">{{positives.length}}</span></span>
      <span @click="select(1, $event)" class="block negative" :class="{'active': selectType===1}">{{desc.negative}}<span class="count">{{negatives.length}}</span></span>
    </div>
    <div class="switch" @click="toggleContent" :class="{'on': onlyContent}">
      <span class="icon-check_circle"></span>
      <span class="text">只看有内容的评价</span>
    </div>
  </div>
</template>

<script>
const POSITIVE = 0;
const NEGATIVE = 1;
const ALL = 2;

export default {
  props: {
    ratings: {
      type: Array,
      default() {
        return [];
      }
    },
    selectType: {
      type: Number,
      default: ALL
    },
    onlyContent: {
      type: Boolean,
      default: true
    },
    desc: {
      type: Object,
      default() {
        return {
          all: '全部',
          positive: '满意',
          negative: '不满意'
        };
      }
    }
  },
  computed: {
    positives() {
      return this.ratings.filter((item) => {
        return item.rateType === POSITIVE;
      });
    },
    negatives() {
      return this.ratings.filter((item) => {
        return item.rateType === NEGATIVE;
      });
    }
  },
  methods: {
    // 是否只显示有内容评论，派发给父级修改onlyContent
    toggleContent() {
      if (!event._constructed) {
        return;
      }
      this.$emit('toggle');
    },
    // 评论类别，派发给父级
    select(type, event) {
      if (!event._constructed) {
        return;
      }
      this.$emit('select', type);
    }
  }
};
</script>

<style lang="less">
@import "../../common/less/index.less";

.ratingselect {
  .rating-type {
    padding: 18px 0;
    margin: 0 18px;
    font-size: 0;
    .border-1px(rgba(7, 17, 27, 0.1));
    .block {
      display: inline-block;
      margin-right: 8px;
      padding: 8px 12px;
      font-size: 12px;
      color: rgb(77, 85, 93);
      line-height: 16px;
      &.active {
        color: #fff;
      }
      .count {
        margin-left: 2px;
        font-size: 8px;
      }
      &.positive {
        background: rgba(0, 160, 220, 0.2);
        &.active {
          background: rgb(0, 160, 220);
        }
      }
      &.negative {
        background: rgba(77, 85, 93, 0.2);
        &.active {
          background: rgb(77, 85, 93);
        }
      }
    }
  }
  .switch {
    padding: 12px 18px;
    font-size: 0;
    color: rgb(147, 153, 159);
    border-bottom: 1px solid rgba(7, 17, 27, 0.1);
    .icon-check_circle {
      font-size: 24px;
      line-height: 24px;
      margin-right: 4px;
    }
    .text {
      vertical-align: top;
      font-size: 12px;
      line-height: 24px;
    }
    &.on {
      .icon-check_circle {
        color: #00c850;
      }
    }
  }
}
</style>
