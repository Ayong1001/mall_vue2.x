<template>
  <swiper>
    <swiper-item v-for="item in banners">
      <a :href="item.link">
        <img :src="item.image" alt="" @load="swiperImgLoad">
      </a>
    </swiper-item>
  </swiper>
</template>


<script>
  import {Swiper, SwiperItem} from 'components/common/swiper'

  export default {
    name: "HomeSwiper",
    props: {
      banners: {
        type: Array,
        default() {
          return []
        }
      }
    },
    components: {
      Swiper,
      SwiperItem
    },
    data(){
      return{
        isLoad:false
      }
    },
    methods:{
      swiperImgLoad(){
        if(!this.isLoad){
          this.$emit('swiperImgLoad')
          this.isLoad=true
        }
      }
    },

  }
</script>

<style scoped>

</style>
