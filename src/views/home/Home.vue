<template>
  <div id="home" class="wrapper">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <tab-control :titles="['流行', '新款', '精选']"
                 @tabClick="tabClick"
                 ref="tabControl1" class="tab-control" v-show="isTabControlFixed"/>

    <scroll class="content"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll"
            :pull-up-load="true" @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImgLoad="swiperImgLoad"/>
      <recommend-view :recommends="recommends"/>
      <feature-view/>
      <tab-control :titles="['流行', '新款', '精选']"
                   @tabClick="tabClick"
                   ref="tabControl2"/>
      <good-list :goods="showGoods"/>
    </scroll>

<!--    <div>呵呵呵呵</div>-->
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
  </div>
</template>

<script>
  import HomeSwiper from './childComps/HomeSwiper'
  import RecommendView from './childComps/RecommendView'
  import FeatureView from './childComps/FeatureView'

  import NavBar from 'components/common/navbar/NavBar'
  import TabControl from 'components/content/tabControl/TabControl'
  import GoodList from 'components/content/goods/GoodsList'
  import Scroll from 'components/common/scroll/Scroll'
  import BackTop from 'components/content/backTop/BackTop'

  import { getHomeMultidata, getHomeGoods } from "network/home"
  import {itemListenerMixin,backTopMixin} from "@/common/mixin";

  export default {
    name: "Home",
    components: {
      HomeSwiper,
      RecommendView,
      FeatureView,
      NavBar,
      TabControl,
      GoodList,
      Scroll,
      BackTop
    },
    mixins:[itemListenerMixin,backTopMixin],
    data() {
      return {
        banners: [],
        recommends: [],
        goods: {
          'pop': {page: 0, list: []},
          'new': {page: 0, list: []},
          'sell': {page: 0, list: []},
        },
        currentType: 'pop',
        tabOffsetTop:0,
        isTabControlFixed:false,
        saveY:0,
      }
    },
    computed: {
      showGoods() {
        return this.goods[this.currentType].list
      }
    },
    created() {
      // 1.请求多个数据
      this.getHomeMultidata()

      // 2.请求商品数据
      this.getHomeGoods('pop')
      this.getHomeGoods('new')
      this.getHomeGoods('sell')


    },
    mounted() {
      //监听商品加载完成,刷新scroll
      this.$bus.$on('itemImageLoad',()=>{
        this.$refs.scroll.refresh()
      })
      //创建完便手动点击一次
      this.tabClick(0)

    },
    activated() {
      this.$refs.scroll.refresh()
      this.$refs.scroll.scrollTo(0, this.saveY,0)
    },
    deactivated() {
      this.saveY=this.$refs.scroll.getScrollY()
      //取消全局事件的监听
      this.$bus.$off('itemImageLoad',this.itemImgListener)
    },
    destroyed() {
    },
    methods: {
      /**
       * 事件监听相关的方法
       */

      tabClick(index) {
        this.currentType=['pop','new','sell'][index]
        this.$refs.tabControl1.currentIndex=index
        this.$refs.tabControl2.currentIndex=index
      },
      backClick() {
        this.$refs.scroll.scrollTo(0, 0)
      },
      contentScroll(position) {
        //决定什么时候显示 回到顶部图标(BackTop)
        this.listenShowBackTop(position)
        this.isTabControlFixed=(-position.y)>this.tabOffsetTop
      },
      loadMore() {
        this.getHomeGoods(this.currentType)
      },
      swiperImgLoad(){
        //获取tabControl的offsetTop,所有的组件都有属性$el用于获取组件中的元素
        this.tabOffsetTop=this.$refs.tabControl2.$el.offsetTop
      },
      /**
       * 网络请求相关的方法
       */
      getHomeMultidata() {
        getHomeMultidata().then(res => {
          // this.result = res;
          this.banners = res.data.banner.list;
          this.recommends = res.data.recommend.list;
        })
      },
      getHomeGoods(type) {
        const page = this.goods[type].page + 1
        getHomeGoods(type, page).then(res => {
          this.goods[type].list.push(...res.data.list)
          this.goods[type].page += 1
          this.$refs.scroll.finishPullUp()
        })
      }
    }
  }
</script>

<style scoped>
  #home {
    /*padding-top: 44px;*/
    height: 100vh;
    position: relative;
  }

  .home-nav {
    background-color: var(--color-tint);
    color: #fff;

    /*position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9;*/
  }

  .content {
    overflow: hidden;
    height: calc(100% - 44px - 49px);
    background-color: #fff;
  }

  .tab-control{
    position: fixed;
    left: 0;
    right: 0;
    z-index: 9;
  }
  /*.fixed{
    position: fixed;
    left: 0;
    right: 0;
    top:44px;
  }*/
  /*.content {*/
    /*height: calc(100% - 93px);*/
    /*overflow: hidden;*/
    /*margin-top: 44px;*/
  /*}*/
</style>
