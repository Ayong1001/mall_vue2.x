<template>
  <div id="category">
    <nav-bar class="nav-bar"><div slot="center">商品分类</div></nav-bar>
    <div class="content">
      <tab-menu :categories="categories"
                @selectItem="selectItem"/>

      <scroll id="tab-content"
              :data="[categoryData]"
              ref="scroll">
        <div>
          <tab-content-category :subcategories="showSubcategory"/>
          <tab-control :titles="['综合', '新品', '销量']"
                       @itemClick="tabClick"/>
          <goods-list v-if="showCategoryDetail" :goods="showCategoryDetail"/>
        </div>
      </scroll>
    </div>
  </div>
</template>

<script>
  import NavBar from '@/components/common/navbar/NavBar'

  import TabMenu from './childComps/TabMenu'
  import TabContentCategory from './childComps/TabContentCategory'


  import TabControl from '@/components/content/tabControl/TabControl'
  import Scroll from '@/components/common/scroll/Scroll'
  import GoodsList from '@/components/content/goods/GoodsList'

  import {getCategory, getSubcategory, getCategoryDetail} from "@/network/category";
  import {POP, SELL, NEW} from "@/common/const"
  import {tabControlMixin} from "@/common/mixin";

  export default {
		name: "Category",
    components: {
		  NavBar,
      TabMenu,
      TabControl,
      Scroll,
      TabContentCategory,
      GoodsList
    },
    mixins: [tabControlMixin],
    data() {
		  return {
		    categories: [],
        categoryData: {
        },
        currentIndex: -1
      }
    },
    mounted() {
      //监听商品加载完成,刷新scroll
      this.$bus.$on('itemImageLoad',()=>{
        this.$refs.scroll.refresh()
      })
    },
    created() {
		  // 1.请求分类数据
      this._getCategory()

    },
    computed: {
		  showSubcategory() {
		    if (this.currentIndex === -1) return {}
        return this.categoryData[this.currentIndex].subcategories
      },
      showCategoryDetail() {
		    if (this.currentIndex === -1) return []
		    return this.categoryData[this.currentIndex].categoryDetail[this.currentType]
      }
    },
    methods: {
		  _getCategory() {
		    getCategory().then(res => {
		      // 1.获取分类数据
		      this.categories = res.data.category.list
			    //console.log(res);
			    // 2.初始化每个类别的子数据
          for (let i = 0; i < this.categories.length; i++) {
            this.categoryData[i] = {
              subcategories: {},
              categoryDetail: {
                'pop': [],
                'new': [],
                'sell': []
              }
            }
          }
          // 3.请求第一个分类的数据
          this._getSubcategories(0)
        })
      },
      _getSubcategories(index) {
        this.currentIndex = index;
		    const mailKey = this.categories[index].maitKey;
        getSubcategory(mailKey).then(res => {
          this.categoryData[index].subcategories = res.data
          this.categoryData = {...this.categoryData}
          this._getCategoryDetail(POP)
          this._getCategoryDetail(SELL)
          this._getCategoryDetail(NEW)
        })
      },
      _getCategoryDetail(type) {
		    // 1.获取请求的miniWallkey
        const miniWallkey = this.categories[this.currentIndex].miniWallkey;
        // 2.发送请求,传入miniWallkey和type
		    getCategoryDetail(miniWallkey, type).then(res => {
		      // 3.将获取的数据保存下来
		      this.categoryData[this.currentIndex].categoryDetail[type] = res
          this.categoryData = {...this.categoryData}
        })
      },
      /**
       * 事件响应相关的方法
       */
      selectItem(index) {
        this._getSubcategories(index)
      }
    }
	}
</script>

<style scoped>
  #category {
    height: 100vh;
  }

  .nav-bar {
    background-color: var(--color-tint);
    font-weight: 700;
    color: #fff;
  }

  .content {
    position: absolute;
    left: 0;
    right: 0;
    top: 44px;
    bottom: 49px;

    display: flex;
  }

  #tab-content {
    overflow: hidden;
    height: 100%;
    flex: 1;
  }
</style>
