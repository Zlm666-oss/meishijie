<template>
    <div class="recipe">
        <!-- v-model="activeName" -->
        <!-- 菜谱分类 start -->
        <el-tabs type="border-card" v-model="c">
            <el-tab-pane 
                :label="item.parent_name" 
                v-for="(item,index) in classify" 
                :key="index"
                :name='item.parent_type'
                >
                <div class="recipe-link">
                    <router-link 
                        :to="{query:{...$router.query,'classify':option.type}}" 
                        v-for="(option,index) in item.list" 
                        :key="index" 
                        :class="{'active':classifyType===option.type}"
                    >
                        {{option.name}}
                    </router-link>
                </div>
            </el-tab-pane>
        </el-tabs>
        <!-- 菜谱分类 end -->
        <h2>家常好味道，给你家一般的温暖</h2>
        <el-container>
            <el-aside width="220px" class="recipe-aside">
                <div class="filter-box">
                    <h4>筛选</h4>
                    <!-- v-model="activeName" -->
                    <!-- 筛选 start -->
                    <el-collapse >
                        <el-collapse-item
                                :title="item.parent_name"
                                v-for="(item,index) in shaixuan"
                                :key="index"
                        >
                            <div class="filter-tags">
                                <el-tag 
                                    type="info"
                                    v-for="(option,index) in item.list"
                                    :key="index"
                                    @click="sx(option)"
                                    :class="{'tag-selected':shaixuanType[option.title]===option.type}"
                                >
                                    {{option.name}}
                                </el-tag>
                            </div>
                        </el-collapse-item>
                    </el-collapse>
                    <!-- 筛选 end -->
                </div>
            </el-aside>



            <el-main class="filter-menus-box">
<!--                <div class="menu-empty">暂时没有过滤出菜谱信息，请选择其他的筛选条件</div>-->
                <menu-card style="min-height: 75%;" :info="list"></menu-card>
                <div style="text-align: right;">
                    <el-pagination
                            style="display: inline-block;"
                            :page-size="5"
                            layout="total, prev, pager, next"
                            :total="total"
                            :current-page.sync="page"
                            @current-change='handlerSelect'
                            >
                    </el-pagination>
                </div>
            </el-main>
        </el-container>
    </div>
</template>
<script>
    import MenuCard from '@/components/menu-card.vue'
    import {getClassify, getProperty, getMenus} from '@/service/api';
    export default {
        components: {MenuCard},
        data() {
            return {
                classify:[],
                classifyType:'1-1',
                c:'1',
                shaixuan:[],
                shaixuanType:'',
                list:[],
                total:0,
                loading:false,
                page:1
            }
        },
        watch: {
            $route: {
                handler(value){
                    const {classify}=value.query
                    this.classifyType=classify
                    this.c=classify[0]
                    this.getMenus()
                },
                immediate:true
            }
        },
        mounted() {
            getClassify().then(({data})=>{
                this.classify=data
            })
            getProperty().then(({data})=>{
                this.shaixuan=data
                const {query}=this.$route
                this.shaixuanType=this.shaixuan.reduce((o,item)=>{
                     o[item.title]=query[item.title] ? query[item.title]: "";
                     return o
                },{})
            })
            
        },
        methods: {
            sx(option){
                const query={...this.$route.query}
                if(this.shaixuanType[option.title]===option.type){
                    this.shaixuanType[option.title]=''
                    delete query[option.title]
                }else{
                    this.shaixuanType[option.title]=option.type
                    query[option.title]=option.type
                }
                this.$router.push({
                    query
                })
            },
            getMenus(){
                const query={...this.$route.query}
                const params={
                    page:query.page||1,
                    classify:query.classify
                }
                delete query.page
                delete query.classify
                if(Object.keys(query).length){
                    params.property={
                        ...query
                    }
                }
                this.loading=true
                let loading=null
                this.$nextTick(()=>{
                    loading=this.$loading({
                        target:'.filter-menus-box',
                        text:'loading...',
                        spinner:'el-icon',
                        background:'rgb(0,0,0,0.7)'
                    })
                })
                getMenus(params).then(({data})=>{
                    if(this.loading) loading.close()
                    this.list=data.list
                    this.total=data.total
                    this.page=data.current_page
                })

            },
            handlerSelect(){
                this.$router.push({
                    query:{
                        ...this.$route.query,
                        page:this.page
                    }
                })
            }
        }
    }
</script>
<style lang="stylus">
    .recipe-link
        font-size 0;
        margin-top 5px

        a
            display inline-block
            font-size 12px
            padding 0px 8px
            height 28px
            line-height 28px

        .active
            background #ff3232
            color #fff

    .recipe
        h2
            text-align center
            line-height 150px

        .el-main
            padding 0

        .filter-box
            background #fff
            padding 10px
            width 100%
            float left
            box-sizing border-box

    .filter-tags
        display flex
        flex-wrap wrap
        justify-content space-around

        .tag-selected
            background-color #ff3232
            color #fff

    .menu-empty
        width 100%
        text-align center
        font-size 20px
</style>

