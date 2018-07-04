<template>
  <div class="star" :class="starType">
    <span v-for="(itemClass, index) in itemClasses" :class="itemClass" class="star-item" :key="index"></span>
  </div>
</template>

<script>
const LENGTH = 5;
const CLS_ON = 'on';
const CLS_OFF = 'off';
const CLS_HALF = 'half';

export default {
  props: {
    size: {
      type: Number
    },
    score: {
      type: Number,
      default: 0
    }
  },
  computed: {
    starType() {
      return 'star-' + this.size;
    },
    itemClasses() {
      var ret = [];
      var score = Math.floor(this.score * 2) / 2;
      var hasDecimal = score % 1 !== 0;
      for (let i = 0; i < score; i++) {
        ret.push(CLS_ON);
      }
      if (hasDecimal) {
        ret.push(CLS_HALF);
      }
      while (LENGTH - ret.length) {
        ret.push(CLS_OFF);
      }
      return ret;
    }
  }
};
</script>

<style lang="less">
@import "../../common/less/index.less";

.star {
  font-size: 0;
  .star-item {
    display: inline-block;
    background-repeat: no-repeat;
  }
  &.star-48 {
    .star-item {
      width: 20px;
      height: 20px;
      margin-right: 22px;
      background-size: 20px 20px;
      &:last-child {
        margin-right: 0;
      }
      &.on {
        .bg-image('star48_on');
      }
      &.half {
        .bg-image('star48_half');
      }
      &.off {
        .bg-image('star48_off');
      }
    }
  }
}
</style>
