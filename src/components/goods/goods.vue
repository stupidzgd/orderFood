<template>
  <div>
    <div class="goods">
      <div class="menu-wrapper" ref="menuWrapper">
        <ul>
          <li v-for="(item, index) in goods" class="menu-item" @click="selectMenu(index, $event)" :class="{'current': currentIndex === index}" ref="menuList" :key="index">
            <span class="text">
              <span v-show="item.type > 0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
            </span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper" ref="foodsWrapper">
        <ul>
          <li v-for="(item, index) in goods" class="food-list" :key="index" ref="foodList">
            <h1 class="title">{{item.name}}</h1>
            <ul>
              <li v-for="(food, index) in item.foods" :key="index" class="food-item" @click="selectFood(food)">
                <div class="icon">
                  <img :src="food.icon" width="57" height="57">
                </div>
                <div class="content">
                  <h2 class="name">{{food.name}}</h2>
                  <p class="desc">{{food.description}}</p>
                  <div class="mes-wrapper">
                    <span class="count">月售{{food.sellCount}}份</span><span class="rating">好评率{{food.rating}}%</span>
                  </div>
                  <div class="price">
                    <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol :food="food" @add="addFood"></cartcontrol>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <shopcart :selectFoods="selectFoods" :deliveryPrice="seller.deliveryPrice" :minPrice="seller.minPrice" ref="shopcart"></shopcart>
    </div>
    <food :food="selectedFood" @add="addFood" ref="food"></food>
  </div>
</template>

<script>
import axios from 'axios';
import BScroll from 'better-scroll';
import shopcart from '@/components/shopcart/shopcart';
import cartcontrol from '@/components/cartcontrol/cartcontrol';
import food from '@/components/food/food';
const debug = process.env.NODE_ENV !== 'production';

export default {
  props: {
    seller: {
      type: Object
    }
  },
  data() {
    return {
      goods: [],
      // 右侧商品栏实时派发出来的y轴滚动距离
      scrollY: 0,
      // 每个类型的商品高度数组，scrollY和数组对别，判断属于哪个区间，从而联动左侧菜单和右侧商品
      listHeight: [],
      selectedFood: {}
    };
  },
  components: {
    shopcart,
    cartcontrol,
    food
  },
  created() {
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];

    const url = debug ? '/api/goods' : 'http://154.8.140.180/sell/api/goods';
    axios.get(url).then((response) => {
      response = response.data;
      if (response.errno === 0) {
        this.goods = response.data;
      }
      this.$nextTick(() => {
        this._initScroll();
        this._calculateHeight();
      });
    });
  },
  computed: {
    currentIndex() {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[i];
        let height2 = this.listHeight[i + 1];
        // 判断当前Y值所处的区间
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          this._followScroll(i);
          return i;
        }
      }
    },
    // 保存被加入到购物车的商品
    selectFoods() {
      let foods = [];
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count > 0) {
            foods.push(food);
          }
        });
      });
      return foods;
    }
  },
  methods: {
    selectMenu(index, event) {
      // event._constructed事件带有这个属性表明是滑动组件触发的click事件，如果不判读click会触发两次，原生一次，better-scroll一次
      if (!event._constructed) {
        return;
      }
      let el = this.$refs.foodList[index];
      // 滚动到菜单对应的商品
      this.foodsScroll.scrollToElement(el, 300, 0, 0);
    },
    selectFood(food) {
      this.selectedFood = food;
      this.$refs.food.show();
    },
    addFood(target) {
      this._drop(target);
    },
    _drop(target) {
      this.$refs.shopcart.drop(target);
    },
    // 初始化better-scroll组件，监听右侧y轴滚动位置
    _initScroll() {
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {
        click: true
      });
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        // 派发click事件，probeType 3派发滚动位置，惯性滚动依然派发
        click: true,
        probeType: 3
      });
      this.foodsScroll.on('scroll', (pos) => {
        this.scrollY = Math.abs(Math.round(pos.y));
      });
    },
    // 保存各类商品在页面的高度
    _calculateHeight() {
      var foodList = this.$refs.foodList;
      let height = 0;
      this.listHeight.push(height);
      for (let i = 0; i < foodList.length; i++) {
        var item = foodList[i];
        height += item.clientHeight;
        this.listHeight.push(height);
      }
    },
    // menu菜单滚动，菜单滚动是跟随右边商品栏滚动触发的
    _followScroll(index) {
      let menuList = this.$refs.menuList;
      let el = menuList[index];
      this.menuScroll.scrollToElement(el, 300, 0, -100);
    }
  }
};
</script>

<style lang="less">
@import "../../common/less/index.less";

.goods {
  display: flex;
  position: absolute;
  top: 174px;
  bottom: 46px;
  width: 100%;
  overflow: hidden;
  .menu-wrapper {
    flex: 0 0 80px;
    width: 80px;
    background: #f3f5f7;
    .menu-item {
      display: table;
      height: 54px;
      width: 56px;
      padding: 0 12px;
      line-height: 14px;
      &.current {
        position: relative;
        z-index: 10;
        margin-top: -1px;
        background: #fff;
        .text {
          font-weight: 700;
          .border-none();
        }
      }
      .text {
        display: table-cell;
        vertical-align: middle;
        font-size: 12px;
        color: rgb(0, 0, 0);
        .border-1px(rgba(7, 17, 27, 0.1));
      }
      .icon {
        display: inline-block;
        vertical-align: top;
        width: 12px;
        height: 12px;
        margin-right: 2px;
        background-size: 12px 12px;
        background-repeat: no-repeat;
        &.decrease {
          .bg-image("decrease_3");
        }
        &.discount {
          .bg-image("discount_3");
        }
        &.guarantee {
          .bg-image("guarantee_3");
        }
        &.invoice {
          .bg-image("invoice_3");
        }
        &.special {
          .bg-image("special_3");
        }
      }
    }
  }
  .foods-wrapper {
    flex: 1;
    .title {
      padding-left: 14px;
      height: 26px;
      font-size: 12px;
      color: rgb(147, 153, 159);
      line-height: 26px;
      border-left: 2px solid #d9dde1;
      background-color: #f3f5f7;
    }
    .food-item {
      position: relative;
      display: flex;
      margin: 18px;
      padding-bottom: 18px;
      .border-1px(rgba(7, 17, 27, 0.1));
      &:last-child {
        margin-bottom: 0;
        .border-none();
      }
      .icon {
        flex: 0 0 57px;
        margin-right: 10px;
      }
      .content {
        flex: 1;
        .name {
          margin: 2px 0 8px 0;
          font-size: 14px;
          color: rgb(7, 17, 27);
          line-height: 14px;
          height: 14px;
        }
        .desc,
        .mes-wrapper {
          font-size: 10px;
          line-height: 10px;
          color: rgb(147, 153, 159);
        }
        .desc {
          margin-bottom: 8px;
          line-height: 12px;
        }
        .mes-wrapper {
          .count {
            margin-right: 12px;
          }
        }
        .price {
          line-height: 24px;
          height: 24px;
          .now {
            margin-right: 8px;
            font-size: 14px;
            color: rgb(240, 20, 20);
            font-weight: 700;
          }
          .old {
            font-size: 10px;
            color: rgb(147, 153, 159);
            text-decoration: line-through;
          }
        }
        .cartcontrol-wrapper {
          position: absolute;
          right: 0;
          bottom: 12px;
        }
      }
    }
  }
}
</style>
