<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物车</div></nav-bar>
     <tab-control :titles = "['流行', '新品', '精选']"  
      @tabClick="tabClick" 
      ref="tabControl" 
      class="tab-control" v-show=" isTabFixed"/>

    <scroll class="content" ref="scroll" :probe-type="3" 
    @scroll="contentScroll" :pull-up-load="true" 
    @pullingUp="loadMore">
      <home-swiper :banners = 'banners' @swiperImageLoad="swiperImageLoad"/>
      <recommend-view :recommends = 'recommends'></recommend-view>
      <feature-view></feature-view>
      <tab-control :titles = "['流行', '新品', '精选']" class="tabcontrol" 
      @tabClick="tabClick" 
      ref="tabControl"/>
      <goods-list :goods = "goods[currentType].list"/>
    </scroll>
    <!-- 监听组件需要加上.native -->
    <back-top @click.native="backClick" v-show="isshow"></back-top>
  </div>
  
</template>

<script>
import NavBar from 'components/common/navBar/NavBar'
import {getHomeMultidata, getHomeGoods} from 'network/home'
import RecommendView from './HomeChildren/RecommendView'
import HomeSwiper from './HomeChildren/HomeSwiper.vue'
import FeatureView from './HomeChildren/FeatureView'
import TabControl from 'components/content/tabControl/TabControl.vue'
import GoodsList from 'components/content/goods/GoodsList.vue'
import Scroll from 'components/common/scroll/Scroll.vue'
import BackTop from '../../components/content/backTop/BackTop.vue'


  export default {
  components: { 
    NavBar,
    HomeSwiper,
    RecommendView,
    FeatureView,
    TabControl,
    GoodsList,
    Scroll,
    BackTop
  },

  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        'pop': {page: 0, list: []},
        'new': {page: 0, list: []},
        'sell': {page: 0, list: []}
      },
      currentType: 'pop',
      isshow: false,
      tabOffsetTop: 0,
      isTabFixed: false,
      saveY: 0
    }
  },

  created() {
    this.getHomeMultidata();

    this.getHomeGoods('pop')
    this.getHomeGoods('new')
    this.getHomeGoods('sell')

   
  },
  mounted() {
    const refresh = this.debounce(this.$refs.scroll.refresh)
     this.$bus.$on('itemImageLoad', () => {
       refresh()
    })

   

  },

  computed: {
    activated() {
      this.$refs.scroll.scrollTo(0, this.saveY, 0)
    },
    deactivted() {
      this.saveY = this.$refs.scroll.scroll.y
    }
  },


  methods: {
    // 网络请求相关的方法
    getHomeMultidata() {
      getHomeMultidata().then(res => {
      this.banners = res.data.banner.list;
      this.recommends = res.data.recommend.list;
      })
    },
    getHomeGoods(type) {
      const page = this.goods[type].page + 1
      getHomeGoods(type, page).then(res => {
         this.goods[type].list.push(...res.data.list)
         this.goods[type].page +=1

         this.$refs.scroll.finishPullUp()
      })
    },
    // 事件监听相关方法
    tabClick(index) {
      switch (index) {
        case 0:
          this.currentType = 'pop'
          break
        case 1:
          this.currentType = 'new'
          break
        case 2:
          this.currentType = 'sell'
          break
      }
    },
    backClick() {
      this.$refs.scroll.scrollTo(0, 0)
    },
    contentScroll(position) {
      this.isshow = (-position.y) > 1000

      this.isTabFixed = (-position.y) > this.tabOffsetTop
    },
    loadMore() {
      //console.log('上拉加载')
      this.getHomeGoods(this.currentType)
    },
    debounce(func, delay) {
      let timer = null
      return function(...args) {
        if (timer) clearTimeout(timer)

        timer = setTimeout(() => {
          func.apply(this, args)
        }, delay)
      }
    },
    swiperImageLoad() {
      this.tabOffsetTop = this.$refs.tabControl.$el.offsetTop
    }
  },
    name: "Home"
  }
</script>

<style scoped>
.home-nav{
  background-color: var(--color-tint);
  color: #fff;
}
#home{
  padding-top: 44px;
  height: 100vh;
  position: relative;

}

.tabcontrol {
  /* position: sticky; */
  top: 44px;
  z-index:  9;
}

.content{
  overflow: hidden;
  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
  right: 0;
}

.tab-control{
  position: relative;
  z-index: 9;
}


</style>
