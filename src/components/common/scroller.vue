<style lang="scss" >
    .wapper {
        /*-webkit-overflow-scrolling: touch;*/
        height: 100%;
        overflow-y: auto;
        box-sizing: border-box;
        position: relative;

        &_content {
            min-height: calc(100% + 1px);
        }

        .swiper_pulldown {
            color: #2176f2;
            top: -60px;
            position: absolute;
            left: calc(50% - 15px);
            width: 30px;
            margin: 0 auto;
            font-size: 25px;
            text-align: center;
            height: 60px;
            overflow: hidden;
            line-height: 60px;
        }

        .swiper_data {
            text-align: center;
            padding-top: 20px;
            padding-bottom: 30px;
            color: #666;
        }

        .swiper_pullload {
            // width: 60px;
            // margin: 0 auto;
			display: block;
			margin-right: 0.15rem;
            font-size: 0.35rem;
            text-align: center;
            height: 60px;
            overflow: hidden;
            line-height: 60px;
            // color: #877ceb;
			color: #666;
			transform-origin: center;
            animation: loading 1s linear infinite;
			display: block;

            &--text {
                display: block;
                width: 100%;
                font-style: normal;
                margin: 0 auto;
                font-size: 25px;
                color: #666;
                text-align: center;
                height: 60px;
                text-align: center;
                overflow: hidden;
                font-size: 14px;
                line-height: 60px;
            }
        }

        .swiper_height {
            height: 0;
            transition: all 0.1s ease-in;
        }

        @keyframes loading {
            0% {
                transform: rotate(0deg);
            }
            100% {
                transform: rotate(360deg);
            }
        }

        .swiper_rote {
            transition: all 0.1s ease-in;
            transform: rotate(180deg);
        }
    }
</style>

<template>
    <div ref="wrapper" class="wapper">
        <div class="wapper_content">
            <!-- 全局icon全部使用svg矢量图 -->
            
			<div class="flex flex-jcac" v-if="!staPullDown">
				<i :class="{'swiper_height':staPullDown}" class="iconfont  swiper_pullload icon-gao-loading2"></i>
				<span style="color: #666; font-size: 0.28rem;">玩命加载中...</span>
			</div>
            <!-- 显示加载中的icon，切换使用svg -->
            <i v-show="staPullDown" :class="{'swiper_rote':hasPullDown}" class="iconfont  swiper_pulldown icon-gao-iconset0413"></i>
            <slot></slot>
            <!-- 上拉加载更多 -->
            <!-- <i v-show="swiper_pullUp" class="iconfont  swiper_pullload icon-gao-loading"></i> -->
			<div class="flex flex-jcac" v-if="swiper_pullUp">
				<i  class="iconfont  swiper_pullload icon-gao-loading2"></i>
				<span style="color: #666; font-size: 0.28rem;">玩命加载中...</span>
			</div>
            <i v-show="!swiper_pullUp&&staPullDown&&pullup&&!swiper_nodata" class="swiper_pullload--text"></i>
            <!-- <i v-show="swiper_nodata" class="iconfont  swiper_data">&#xe668;</i> -->
            <empty v-if="empty"></empty>
            <no-more v-else-if="swiper_nodata"></no-more>
        </div>
    </div>
</template>

<script>
    import BScroll from 'better-scroll'

    export default {
        data() {
            return {
                staPullDown: true,
                hasPullDown: false,
            }
        },
        props: {
            /* 是否可以下拉加载 */
            swiper_pullUp: {
                type: Boolean,
                default: false,
            },
            swiper_nodata: {
                type: Boolean,
                default: false,
            },
            empty:{
                type:Boolean,
                default:false
            },
            /**
             * 1 滚动的时候会派发scroll事件，会截流。
             * 2 滚动的时候实时派发scroll事件，不会截流。
             * 3 除了实时派发scroll事件，在swipe的情况下仍然能实时派发scroll事件
             */
            probeType: {
                type: Number,
                default: 1
            },
            /**
             * 点击列表是否派发click事件
             */
            click: {
                type: Boolean,
                default: true
            },
            /**
             * 是否开启横向滚动
             */
            scrollX: {
                type: Boolean,
                default: false
            },
            /**
             * 是否派发滚动事件
             */
            listenScroll: {
                type: Boolean,
                default: true
            },
            /**
             * 列表的数据
             */
            data: {
                type: Array,
                default: null
            },
            /**
             * 是否派发滚动到底部的事件，用于上拉加载
             */
            pullup: {
                type: Boolean,
                default: false
            },
            /**
             * 是否派发顶部下拉的事件，用于下拉刷新
             */
            pulldown: {
                type: Boolean,
                default: false
            },
            /**
             * 是否派发列表滚动开始的事件
             */
            beforeScroll: {
                type: Boolean,
                default: true
            },
            /**
             * 当数据更新后，刷新scroll的延时。
             */
            refreshDelay: {
                type: Number,
                default: 20
            }
        },
        mounted() {
            // 保证在DOM渲染完毕后初始化better-scroll
            setTimeout(() => {
                this._initScroll();
            }, 20)
        },
        methods: {
            _initScroll() {
                if (!this.$refs.wrapper) {
                    return
                }
                // better-scroll的初始化
                this.scroll = new BScroll(this.$refs.wrapper, {
                    probeType: this.probeType,
                    click: this.click,
                    scrollX: this.scrollX
                })

                // 是否派发滚动事件
                if (this.listenScroll) {
                    this.scroll.on('scroll', (pos) => {
                        if (pos.y > 50) {
                            this.hasPullDown = true;
                        }
                        this.$emit('scroll', pos)
                    })
                }

                // 是否派发滚动到底部事件，用于上拉加载
                if (this.pullup) {
                    this.scroll.on('scrollEnd', (pos) => {
                        // 滚动到底部
                        if (this.scroll.y <= (this.scroll.maxScrollY)) {
                            this.$emit('scrollToEnd')
                        }
                    })
                }

                // 是否派发顶部下拉事件，用于下拉刷新
                if (this.pulldown) {
                    this.scroll.on('touchEnd', (pos) => {
                        // 下拉动作
                        if (pos.y > 50) {
                            this.staPullDown = false;
                            this.hasPullDown = false;
                            this.$emit('pulldown');
                            setTimeout(() => {
                                this.staPullDown = true;
                            }, 500);
                        }
                    })
                }

                // 是否派发列表滚动开始的事件
                if (this.beforeScroll) {
                    this.scroll.on('beforeScrollStart', () => {
                        this.$emit('beforeScroll');
                    })
                }
            },
            disable() {
                // 代理better-scroll的disable方法
                this.scroll && this.scroll.disable()
            },
            enable() {
                // 代理better-scroll的enable方法
                this.scroll && this.scroll.enable()
            },
            refresh() {
                // 代理better-scroll的refresh方法
                this.scroll && this.scroll.refresh()
            },
            scrollTo() {
                // 代理better-scroll的scrollTo方法
                this.scroll && this.scroll.scrollTo.apply(this.scroll, arguments)
            },
            scrollToElement() {
                // 代理better-scroll的scrollToElement方法
                this.scroll && this.scroll.scrollToElement.apply(this.scroll, arguments)
            }
        },
        watch: {
            // 监听数据的变化，延时refreshDelay时间后调用refresh方法重新计算，保证滚动效果正常
            // 此处监听了data的数据变化，实际上数据可能一个类数组的对象列表，同时应该考虑到页面多数据的情况
            data() {
                setTimeout(() => {
                    this.refresh()
                }, this.refreshDelay)
            }
        }
    }
</script>
