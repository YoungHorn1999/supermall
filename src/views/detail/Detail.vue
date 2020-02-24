<template>
    <div id="detail">
        <detail-nav-bar class="detail-nav" @titleClick="titleClick"/>
        <scroll class="content" ref="scroll">
            <detail-swiper :top-images="topImages"/>
            <detail-base-info :goods="goods"/>
            <detail-shop-info :shop="shop"/>
            <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"/>
            <detail-param-info ref="params" :param-info="paramInfo"/>
            <detail-comment-info ref="comment" :comment-info="commentInfo"/>
            <goods-list ref="recommend" :goods="recommends"/>
        </scroll>
    </div>
</template>

<script>
    import DetailNavBar from './childComps/DetailNavBar'
    import DetailSwiper from './childComps/DetailSwiper'
    import DetailBaseInfo from './childComps/DetailBaseInfo'
    import DetailShopInfo from './childComps/DetailShopInfo'
    import DetailGoodsInfo from './childComps/DetailGoodsInfo'
    import DetailParamInfo from './childComps/DetailParamInfo'
    import DetailCommentInfo from './childComps/DetailCommentInfo'

    import Scroll from 'components/common/scroll/Scroll'
    import GoodsList from 'components/content/goods/GoodsList'

    import {getDetail, Goods, Shop, GoodsParam, getRecommend} from "network/detail";
    import {debounce} from "common/utils";
    import {itemListenerMixin} from "common/mixin";

    export default {
        name: "Detail",
        components: {
            DetailNavBar,
            DetailSwiper,
            DetailBaseInfo,
            DetailShopInfo,
            DetailGoodsInfo,
            DetailParamInfo,
            DetailCommentInfo,
            Scroll,
            GoodsList
        },
        mixins: [itemListenerMixin],
        data() {
            return {
                iid: null,
                topImages: [],
                goods: {},
                shop: {},
                detailInfo: {},
                paramInfo: {},
                commentInfo: {},
                recommends: [],
                themeTopYs: [],
                getThemeTopY: null
                
            }
        },
        created() {
            //1.保存传入的iid
            this.iid = this.$route.params.iid

            //2.根据iid请求详情数据
            getDetail(this.iid).then(res => {
                //1.获取顶部的图片轮播数据
                // console.log(res);
                const data = res.result;
                this.topImages = data.itemInfo.topImages

                //2.获取商品信息
                this.goods = new Goods(data.itemInfo, data.columns, data.shopInfo.services)

                //3.创建店铺信息的对象
                this.shop = new Shop(data.shopInfo)

                //4.保存商品的详情数据
                this.detailInfo = data.detailInfo;

                //5.获取参数信息
                this.paramInfo = new GoodsParam(data.itemParams.info, data.itemParams.rule)

                //6.取出评论的信息
                if (data.rate.cRate !==0) {
                    this.commentInfo = data.rate.list[0]
                }

                // this.$nextTick(() => {
                //     //根据最新的数据，对应的DOM是已经被渲染出来
                //     //但是图片依然是没有加载完（目前获取到offsetTop不包含其中的图片）
                //     //offsetTop值不对的时候，一般都是因为图片的问题
                //     this.themeTopYs = []

                //     this.themeTopYs.push(0);
                //     this.themeTopYs.push(this.$refs.params.$el.offsetTop);
                //     this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
                //     this.themeTopYs.push(this.$refs.recommend.$el.offsetTop);

                //     console.log(this.themeTopYs);
                // })

                
            })

            //3.请求推荐数据
            getRecommend().then(res => {
                this.recommends = res.data.list
            })

            //4.给getThemeTopY赋值
            this.getThemeTopY = debounce(() => {
                 this.themeTopYs = []
                    this.themeTopYs.push(0);
                    this.themeTopYs.push(this.$refs.params.$el.offsetTop);
                    this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
                    this.themeTopYs.push(this.$refs.recommend.$el.offsetTop);

                    console.log(this.themeTopYs);
            })
        },
        methods: {
            imageLoad() {
                this.$refs.scroll.refresh()

                this.getThemeTopY()
               
            },
            titleClick(index) {
                console.log(index);
                this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 200)
            }
        },
        mounted() {
            
        },
        updated() {
            


        },
        destroyed() {
            this.$bus.$off('itemImgLoad', this.itemImgListener )
        }
    }
</script>

<style scoped>
    #detail {
        position: relative;
        z-index: 9;
        background-color: #fff;
        height: 100vh;
    }

    .content {
        height: calc(100% - 44px);
    }

    .detail-nav {
        position: relative;
        z-index: 9;
        background-color: #fff;
    }
</style>