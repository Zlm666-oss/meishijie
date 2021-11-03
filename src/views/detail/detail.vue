<template>
  <div class="menu-detail">
    <detail-header :info='info'></detail-header>
    <detail-content :info='info'></detail-content>
    <Comment :info='info'></Comment>
  </div>
</template>
<script>
import DetailHeader from './detail-header'
import DetailContent from './detail-content'
import Comment from './comment'
import {menuInfo} from '@/service/api';
export default {
  components: {DetailHeader, DetailContent, Comment},
  data(){
    return {
      info:{
        properties_show:{},
        raw_material:{
          accessories_material:{},
          main_material:{}
        },
        userInfo:{}
      }
    }
  },
  watch:{
    $route:{
      async handler(value){
        const {menuId}=value.query
        const {data}=await menuInfo({menuId})
        this.info=data.info
      },
      immediate:true
    }
  }
}
</script>
