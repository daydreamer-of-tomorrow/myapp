<template>
    <div>
        <!-- 固定定位 -->
        <div class="fixed-bar">
            <div class="reutrn-box">
                <div class="bar-reutrn" @click="returnHandler()"></div>
            </div>
            <div class="bar-ct">
                <!-- 返回箭头 -->
                <span>商品评价</span>
            </div>
        </div>
        <!-- 商品评价 -->
        <scroll-view scroll-y="true" :style="{height:safeHeight + 'px'}" lower-threshold="170"
            @scrolltolower="scrollTolowerHandler()">
            <div class="comment-box" v-if="Object.keys(commentData).length != 0">
                <div class="cb">
                    <div class="cb-l">评价</div>
                    <div class="cb-r">排序</div>
                </div>
                <div class="com-box" v-for="(item,index) in commentData" :key="index">
                    <div class="hpn">
                        <div class="head-portrait">
                            <image :src="`http://www.zkt-it.com:5050/pic/comment/${item.avatar}?format=950w_950h_1e_0l`"
                                mode=""></image>
                        </div>
                        <div>
                            <div class="nick-name">
                                {{item.nickName}}
                            </div>
                            <div class="star" :style="{'background-position': -(5 - item.score) * 16 + 'px 0px'}">
                            </div>
                        </div>
                        <div class="pay-time">{{item.publishTimeStr}}</div>
                    </div>
                    <div class="content">
                        {{item.content}}
                    </div>
                    <div class="pics">
                        <div v-for="(item2,index2) in item.pics" :key="index2">
                            <image :src="`http://www.zkt-it.com:5050/pic/comment/${item2}?format=950w_950h_1e_0l`"
                                mode="aspectFill" @click="bigPics(item.pics, item2)"></image>
                        </div>
                    </div>
                    <div class="pay-color-model">
                        {{item.commodityInfo.charaterDesc2}},{{item.commodityInfo.charaterDesc1}}
                    </div>
                </div>
            </div>
        </scroll-view>

    </div>
</template>

<script>
    let page = 1;
    // 节流锁
    let lock = true;
    export default {
        onLoad(obj) {
            console.log(obj);
            wx.getSystemInfo({
                success: (e) => {
                    console.log(e);
                    this.safeHeight = e.safeArea.height - e.statusBarHeight;
                }
            });
            this.detailSKU = obj.sku;
            this.cluster = obj.cluster;
            this.loadComment();
        },
        data() {
            return {
                detailSKU: '',
                cluster: '',
                commentData: [],
                safeHeight: 0
            }
        },
        methods: {
            // 加载评论
            loadComment() {
                wx.showLoading({
                    title: '加载中...',
                });
                wx.request({
                    url: 'http://www.zkt-it.com:5050/comment/' + this.cluster + '/' + this.detailSKU +
                        '?page=' + page,
                    success: (data) => {
                        console.log(data.data);
                        this.commentData = [...this.commentData, ...data.data];
                        wx.hideLoading();
                        lock = true;
                    }
                })
            },
            // 点小图看大图
            bigPics(pics, nowPic) {
                let urls = pics.map(item =>
                    `http://www.zkt-it.com:5050/pic/comment/${item}?format=950w_950h_1e_0l`);
                let current = `http://www.zkt-it.com:5050/pic/comment/${nowPic}?format=950w_950h_1e_0l`;
                wx.previewImage({
                    urls,
                    current
                })
            },
            // 加载后面的评论
            scrollTolowerHandler() {
                if (lock) {
                    page++;
                    this.loadComment();
                    lock = false;
                }

            },
            // 返回箭头
            returnHandler() {
                wx.navigateBack({
                    delta: 1,
                    success: () => {
                        console.log("返回成功");
                    }
                })
            }
        },
    }
</script>

<style scoped>
    /* 固定 */
    .fixed-bar {
        position: fixed;
        top: 0;
        left: 0;
        width: 750rpx;
        height: 90px;
        background-color: #f4f4f4;
        z-index: 1;
        overflow: hidden;
    }
    /* 返回箭头盒子 */
    .fixed-bar .reutrn-box{
        width: 750rpx;
        height: 60px;
        position: relative;
    }
    /* 返回箭头 */
    .fixed-bar .reutrn-box .bar-reutrn {
        height: 20px;
        width: 20px;
        background-image: url(../../static/return.png);
        background-repeat: no-repeat;
        background-size: 100% 100%;
        position: absolute;
        left: 15rpx;
        bottom: -25px;
    }
    .fixed-bar .bar-ct {
        width: 750rpx;
        height: 30px;
    }
    .fixed-bar .bar-ct span{
        display: block;
        height: 30px;
        line-height: 30px;
        font-size: 16px;
        text-align: center;
    }

    /* 评论 */
    .comment-box {
        width: 750rpx;
        padding-top: 120px;
        background-color: rgba(218, 216, 216, .3);
    }

    .comment-box .cb {
        width: 750rpx;
        margin: 20px auto;
        background-color: white;
        border-radius: 15px;
        box-sizing: border-box;
        font-size: 16px;
        padding: 0 8px 0;
        display: flex;
        overflow: hidden;
    }

    .comment-box .cb .cb-l {
        color: #696978;
    }

    /* 排序 */
    .comment-box .cb .cb-r {
        margin-left: auto;
        color: #696978;
        text-decoration: line-through;
    }

    .comment-box .com-box {
        padding: 20px 8px 0;
        width: 750rpx;
        margin: 20px auto;
        background-color: white;
        box-sizing: border-box;
        border-radius: 15px;
        overflow: hidden;
    }

    .comment-box .com-box .hpn {
        display: flex;
        margin-bottom: 10px;
        align-items: center;
        overflow: hidden;
    }

    .comment-box .com-box .hpn .head-portrait {
        height: 60rpx;
        width: 60rpx;
        margin-right: 10px;
    }

    .comment-box .com-box .hpn .head-portrait image {
        height: 60rpx;
        width: 60rpx;
        border-radius: 50%;
    }

    .comment-box .com-box .hpn .nick-name {
        font-size: 13px;
        margin-bottom: 4px;
    }

    .comment-box .com-box .hpn .star {
        width: 80px;
        height: 14px;
        background-image: url(../../static/star.png);
        background-repeat: no-repeat;
    }

    /* 购买时间 */
    .comment-box .com-box .hpn .pay-time {
        margin-left: auto;
        font-size: 12px;
    }

    .comment-box .com-box .content {
        font-size: 14px;
        text-align: justify;
        margin-bottom: 10px;
    }

    .comment-box .com-box .pics {
        display: flex;
        width: 750rpx;
        height: 120rpx;
    }

    .comment-box .com-box .pics image {
        margin-right: 10px;
        width: 120rpx;
        height: 120rpx;
    }

    .comment-box .com-box .pay-color-model {
        margin-top: 10px;
        font-size: 12px;
        color: #999999;
        padding: 0 0 20px;
    }
</style>
