<template>
  <div>
    <v-header :seller="seller"></v-header>
    <div class="tab">
      <div class="tab-item">
        <router-link to="/goods">商品</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/ratings">评论</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/seller">商家</router-link>
      </div>
    </div>
    <!-- 缓存路由，避免每次切换路由都重新加载 -->
    <keep-alive>
      <router-view :seller="seller"/>
    </keep-alive>
  </div>
</template>

<script>
import header from '@/components/header/header';
import axios from 'axios';
import {urlParse} from '@/common/js/util';
const ERR_OK = 0;
const debug = process.env.NODE_ENV !== 'production';

export default {
  data() {
    return {
      seller: {
        id: (() => {
          // 解析查询字符串的id值
          let queryParam = urlParse();
          return queryParam.id;
        })()
      }
    };
  },
  components: {
    'v-header': header
  },
  created() {
    const url = debug ? '/api/seller' : 'http://zgd666.cc/sell/api/seller';
    // const url = debug ? '/api/seller' : 'http://172.26.252.15:9000/api/seller';
    axios.get(url + '?id=' + this.seller.id).then((response) => {
      response = response.data;
      if (response.errno === ERR_OK) {
        // 把id属性合并到ajax请求回来的seller对象中
        this.seller = Object.assign({}, this.seller, response.data);
      }
    });
  }
};
</script>

<style lang="less">
@import './common/less/index.less';

.tab {
  display: flex;
  height: 40px;
  line-height: 40px;
  overflow: hidden;
  .border-1px(rgba(7, 17, 27, 0.1));
  .tab-item {
    flex: 1;
    text-align: center;
    & > a {
      display: block;
      font-size: 14px;
      color: rgb(77, 85, 93);
      &.active {
        color: rgb(240, 20, 20);
      }
    }
  }
}
</style>
