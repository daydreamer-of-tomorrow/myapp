<template>
    <div>
        <scroll-view scroll-y="true" :style="{height: windowHeight + 'px'}" :scroll-with-animation="animation"
            :scroll-into-view="moveToPosition" @scroll="scrollChangeHandler($event)">
            <!-- 固定定位 -->
            <div class="fixed-bar" :class="{'bar-o':scrollTop > 170,'bar-p':scrollTop > 10}">    
                <div class="bar-ct">
                    <!-- 返回箭头 -->
                    <div class="bar-reutrn" @click="returnHandler()"></div>
                    <span @click="moveToPosition = 'go-top'">商品</span>
                    <span @click="moveToPosition = 'go-com-box'">评价</span>
                    <span @click="moveToPosition = 'go-pra-box'">详情</span>
                    <span @click="moveToPosition = 'go-recommend'">推荐</span>
                </div>
            </div>
            <div class="details">
                <!-- 如果不写 v-if="allData.pics"，上来会是1/undefined。 -->
                <div id="go-top" class="swiper-box" v-if="allData.pics">
                    <!-- 轮播图 -->
                    <swiper class="swiper-box" indicator-dots="true" indicator-active-color="red" circular="true"
                        :duration="duration" @change="currentHandler($event)" :current="nowIndex"
                        indicator-color="white">
                        <block :class="{transition: nowIndex == 0}" v-for="(item,index) in allData.pics" :key="index">
                            <swiper-item>
                                <image :src="`http://www.zkt-it.com:5050/pic/product/${item}?format=900w_900h_1e_0l`"
                                    mode=""></image>

                            </swiper-item>
                        </block>
                    </swiper>
                    <div class="page-pt">
                        {{nowIndex + 1}} / {{allData.pics.length}}
                    </div>
                </div>
                <!-- 这里不能写v-if="allData",那样一样会在Ajax拉取之前显示undefined,因为allData是一个空对象，一样会认为是true，因而要写v-if="Object.keys(allData).length != 0" -->
                <div class="nps-box" v-if="Object.keys(allData).length != 0">
                    <div class="p-name">
                        {{allData.pName}}
                    </div>
                    <div class="price">
                        <span>￥</span>{{allData.price}}
                    </div>
                    <div class="shop-name">
                        {{allData.shopName}}
                    </div>
                </div>

                <!-- 商品颜色和型号 -->
                <div id="go-com-box" class="color-model" v-if="allData.vo">
                    <div class="cm-son">
                        <div class="cm-c">颜色</div>
                        <div class="cm-for"
                            :class="{'bind-color': colorId == item.characterValueId, disabled:disabledHandler(item.characterValueId, versionId)}"
                            v-for="(item,index) in allData.vo.colorList" :key="index"
                            @click="bindColorHandler(item.characterValueId)">
                            {{item.characterValueName}}
                        </div>
                    </div>
                    <div class="cm-son">
                        <div class="cm-m">型号</div>
                        <div class="cm-for"
                            :class="{'bind-color': versionId == item.characterValueId, disabled:disabledHandler(colorId, item.characterValueId)}"
                            v-for="(item,index) in allData.vo.versionList" :key="index"
                            @click="bindVersionHandler(item.characterValueId)">
                            {{item.characterValueName}}
                        </div>
                    </div>
                </div>

                <!-- 商品评价 -->
                <div id="go-pra-box" class="comment-box" v-if="Object.keys(allData).length != 0">
                    <div class="cb">
                        <div>评论</div>
                        <div class="cb-r" @click="loadMoreComment()">加载更多<span>&gt;&gt;</span></div>
                    </div>
                    <div class="com-box" v-for="(item,index) in commentData" :key="index">
                        <div class="hpn">
                            <div class="head-portrait">
                                <image
                                    :src="`http://www.zkt-it.com:5050/pic/comment/${item.avatar}?format=950w_950h_1e_0l`"
                                    mode=""></image>
                            </div>
                            <div>
                                <div class="nick-name">
                                    {{item.nickName}}
                                </div>
                                <div class="star" :style="{'background-position': -(5 - item.score) * 16 + 'px 0px'}">
                                </div>
                            </div>
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

                    </div>
                </div>

                <!-- 商品详情 -->
                <div class="shop-particulars" v-if="particularsPics.length != 0">
                    <div class="minutiae">详情</div>
                    <div class="sp-pics" v-for="(item,index) in particularsPics" :key="index">
                        <image :src="`http://www.zkt-it.com:5050/pic/product/${item}?format=950w_950h_1e_0l`"
                            mode="widthFix"></image>
                    </div>
                </div>
                <!-- 推荐 ,简单写了下,没写点击跳转页面-->
                <div id="go-recommend" class="recommend" v-if="Object.keys(allData).length != 0">
                    <div class="rec-w">推荐</div>
                    <div class="goods-flax">
                        <div class="goods" v-for="item in goods" :key="item.SKU">
                            <div class="">
                                <!-- 反引号，动态src -->
                                <image
                                    :src="`http://www.zkt-it.com:5050/pic/product/${item.thumbnail}?format=450w_450h_1e_0l`"
                                    mode=""></image>
                            </div>
                            <div class="" style="font-size: 12px;padding-left: 10rpx;">
                                {{item.pName}}
                            </div>
                            <div class="" style="color: orange;font-size: 14px;padding-left: 10rpx;">
                                {{item.price}}元
                            </div>
                    </div>
                    
                    </div>
                </div>
            </div>
        </scroll-view>
    </div>
</template>

<script>
    // 拉取评论的节流锁
    let commentLock = true;
    // 拉取详情的节流锁
    let particularsLock = true;
    // 拉取推荐的节流锁
    let recommendLock = true;
    export default {
        onLoad(obj) {
            wx.getSystemInfo({
                // 窗口高
                success: (e) => {
                    console.log(e);
                    this.windowHeight = e.safeArea.height;
                }
            });
            // console.log(obj);
            this.detailSKU = obj.sku;
            this.keyword = obj.keyword;
            // this.detailSKU = '12345952857_0000000000';
            // 页面跳转开锁
            commentLock = true;
            particularsLock = true;
            recommendLock = true;
            this.loadData();
        },
        data() {
            return {
                // sku
                detailSKU: '',
                // 所有数据
                allData: {},
                // 评论数据
                commentData: [],
                // 当前处于第几张图
                nowIndex: 0,
                // 颜色id
                colorId: "",
                // 版本id
                versionId: "",
                // 窗口高
                windowHeight: 0,
                // 向上滚动高度
                scrollTop: 0,
                // 点击跳转去指定位置
                moveToPosition: '',
                // 刷新间隔
                duration: 500,
                // 详情图片
                particularsPics: [],
                // 跳转时无过渡
                animation: false,
                // 推荐数据数组
                goods: [],
                keyword:''
            }
        },
        methods: {
            loadData() {
                // 开菊花图
                wx.showLoading({
                    title: "加载中...",
                });
                wx.request({
                    url: 'http://www.zkt-it.com:5050/product-detail/' + this.detailSKU,
                    success: data => {
                        console.log(data.data);
                        this.allData = data.data;
                        // 用从project跳转过来的sku推当前的颜色型号
                        let o = this.allData.vo.mappingList.filter(item => item[2] == this.detailSKU)
                        if (o.length == 1) {
                            this.colorId = o[0][0];
                            this.versionId = o[0][1];
                        }
                        // 关菊花图
                        wx.hideLoading();
                        // 重新加载时让轮播图每张过渡时长为500ms
                        this.duration = 500;
                    }
                })
            },
            // @scroll,得到页面滚动时当前页面据顶部的高度
            scrollChangeHandler(e) {
                // console.log(e.detail.scrollTop);
                this.scrollTop = e.detail.scrollTop;
                // 只要一滚动页面就让moveToPosition(跳转去指定位置)为空
                this.moveToPosition = '';
                // 懒加载,当页面据顶部250时加载评论
                if (commentLock && e.detail.scrollTop > 250) {
                    // 关锁,一个页面只加载一次,后面换颜色型号不再加载
                    commentLock = false;
                    this.loadComment();
                }
                // 懒加载,当页面据顶部450时加载详情
                if (particularsLock && e.detail.scrollTop > 450) {
                    // 关锁,一个页面只加载一次,后面换颜色型号不再加载
                    particularsLock = false;
                    this.loadParticulars();
                }
                // 懒加载,当页面据顶部850时加载推荐
                if (recommendLock && e.detail.scrollTop > 850) {
                    // 关锁,一个页面只加载一次,后面换颜色型号不再加载
                    recommendLock = false;
                    this.loadRecommend();
                }
            },
            // 轮播图,当前在第几张图片
            currentHandler(e) {
                // console.log(e.detail);
                this.nowIndex = e.detail.current;
            },
            // 切换颜色
            bindColorHandler(c) {
                // 没该种搭配颜色禁点
                if (this.disabledHandler(c, this.versionId)) {
                    // 没有该种颜色型号搭配弹出提示
                    wx.showToast({
                        title: '亲！！没有该种搭配的货了！！',
                        icon: "none"
                    });
                    return;
                }
                // 得到颜色编码
                this.colorId = c;
                // 得到该种搭配的数组
                let reslut = this.allData.vo.mappingList.filter(item => item[0] == c && item[1] == this.versionId);
                // console.log(reslut[0][2]);
                this.detailSKU = reslut[0][2];
                // 切换时图片归1,类似于页面归一
                this.nowIndex = 0;
                // 页面归一时无过渡效果
                this.duration = 0;
                this.loadData();
            },
            // 切换型号
            bindVersionHandler(v) {
                // 没该种搭配颜色禁点
                if (this.disabledHandler(this.colorId, v)) {
                    // 没有该种颜色型号搭配弹出提示
                    wx.showToast({
                        title: '亲！！没有该种搭配的货了！！',
                        icon: "none"
                    });
                    return;
                }
                // 得到型号编码
                this.versionId = v;
                // 得到该种搭配的数组
                let reslut = this.allData.vo.mappingList.filter(item => item[0] == this.colorId && item[1] == v);
                // console.log(reslut[0][2]);
                // 切换时图片归1,类似于页面归一
                this.nowIndex = 0;
                this.detailSKU = reslut[0][2];
                // 页面归一时无过渡效果
                this.duration = 0;
                // 拉数据
                this.loadData();
            },
            // 删除线事件
            disabledHandler(colorId, versionId) {
                // let o = this.allData.vo.mappingList.filter(item =>item[0] == colorId && item[1] == versionId)
                // console.log(o);
                // if(o.length == 0){
                //     return true;
                // }
                // 开始写的时候忘了some了,后面跟老师的比对的时候,才想起,some得到的是布尔值
                return !this.allData.vo.mappingList.some(item => item[0] == colorId && item[1] == versionId);
            },
            // 加载评论
            loadComment() {
                wx.request({
                    url: 'http://www.zkt-it.com:5050/comment/' + this.allData.cluster + '/' + this.detailSKU,
                    success: (data) => {
                        console.log(data.data);
                        // 在当前页面只显示三条评论
                        this.commentData = data.data.slice(0, 3);
                    }
                })
            },
            // 点小图看大图
            bigPics(pics, nowPic) {
                // 得到pics数组
                let urls = pics.map(item => `http://www.zkt-it.com:5050/pic/comment/${item}?format=950w_950h_1e_0l`);
                // 得到当前图片
                let current = `http://www.zkt-it.com:5050/pic/comment/${nowPic}?format=950w_950h_1e_0l`;
                wx.previewImage({
                    // k,v一致省略v
                    urls,
                    current
                })
            },
            // 加载更多评论
            loadMoreComment() {
                // 跳转页面并将参数cluster和sku传递过去,语法类get请求
                wx.navigateTo({
                    url: '../morecomment/morecomment?cluster=' + this.allData.cluster + '&sku=' + this
                        .detailSKU,
                })
            },
            // 加载产品详情
            loadParticulars() {
                wx.request({
                    url: 'http://www.zkt-it.com:5050/product-detail-pic-description/' + this.detailSKU,
                    success: (data) => {
                        // console.log(data.data);
                        this.particularsPics = data.data;
                    }
                })
            },
            // 加载推荐产品
            loadRecommend() {
                // 拉Ajax
                wx.request({
                    url: "http://www.zkt-it.com:5050/product-search?keyword=" + this.keyword + "&page=1&pagesize=10",
                    success: (data) => {
                        // console.log(data.data.goods);
                        this.goods = data.data.goods;
                    }
                })
            },
            // 返回箭头
            returnHandler(){
                // 测试返回
                // let pages = getCurrentPages();
                // console.log(pages);
                // let prevPage = pages[pages.length-2];
                // console.log(prevPage);
                wx.navigateBack({
                     delta: 1,
                     success:()=>{
                         console.log("返回成功");
                     }
                })
            }
        },
    }
</script>

<style scoped>
    /* 固定 定位*/
    .fixed-bar {
        position: fixed;
        top: 0;
        left: 0;
        width: 750rpx;
        height: 120px;
        background-color: #f4f4f4;
        z-index: 1;
        display: flex;
        justify-content: center;
        align-items: flex-end;
        opacity: 0;
    }

    /* 显示情况 */
    .fixed-bar.bar-p {
        opacity: .1;
        transition: opacity .1s ease 0s;
    }

    .fixed-bar.bar-o {
        opacity: 1;
        transition: opacity .5s ease 0s;
    }

    .fixed-bar .bar-ct {
        padding: 0 0 10px;
        position: relative;
    }
    /* 返回箭头 */
    .fixed-bar .bar-ct .bar-reutrn{
        height: 20px;
        width: 20px;
        background-image: url(../../static/return.png);
        background-repeat: no-repeat;
        background-size: 100% 100%;
        position: absolute;
        left: -180rpx;
        bottom: 10px;
    }
    /* 固定定位里面的内容间距 */
    .fixed-bar .bar-ct span {
        margin-right: 20px;
    }

    .fixed-bar .bar-ct span:last-child {
        margin-right: 0;
    }

    /* 页面大盒子 */
    .details {
        width: 750rpx;
        height: 2000px;
        background: linear-gradient(180deg, rgba(255, 255, 255, .3), rgba(218, 216, 216, .3));
    }

    /* 页面轮播图 */
    .details .swiper-box {
        width: 750rpx;
        height: 750rpx;
        position: relative;
    }

    /* 图片宽高 */
    .details .swiper-box image {
        width: 750rpx;
        height: 750rpx;
    }

    /* 轮播图当前显示第几张图 */
    .details .swiper-box .page-pt {
        position: absolute;
        bottom: 40rpx;
        right: 20rpx;
        width: 80rpx;
        height: 40rpx;
        line-height: 40rpx;
        font-size: 20rpx;
        text-align: center;
        color: rgba(255, 255, 255, .7);
        background-color: rgba(0, 0, 0, .3);
        border-radius: 5px;
    }

    /* pName,price */
    .details .nps-box {
        width: 720rpx;
        margin: 20px auto;
        background-color: white;
        border-radius: 15px;
        overflow: hidden;
    }

    .details .nps-box .p-name {
        font-weight: bold;
        font-size: 16px;
        padding: 20px 8px 5px;
    }

    /* 价格 */
    .details .nps-box .price {
        color: red;
        padding: 5px 8px;
        margin-top: 8px;
        font-size: 20px;
        border-top: 1px solid #ffeeef;
        border-bottom: 1px solid #ffeeef;
    }

    .details .nps-box .price span {
        font-size: 14px;
    }

    /* 苏宁自营 */
    .details .nps-box .shop-name {
        padding: 5px 8px 20px;
        font-size: 12px;
    }

    /* 颜色和型号 */
    .details .color-model {
        width: 720rpx;
        margin: 20px auto;
        background-color: white;
        border-radius: 15px;
        padding-top: 20px;
        overflow: hidden;
    }

    .details .color-model .cm-son {
        margin-bottom: 20px;
        padding: 0px 8px 0px;
        overflow: hidden;
    }

    .details .color-model .cm-son .cm-c,
    .details .color-model .cm-son .cm-m {
        font-size: 12px;
    }

    .details .color-model .cm-son .cm-for {
        display: inline-block;
        margin-top: 10px;
        margin-right: 15px;
        background-color: #f7f7f7;
        font-size: 13px;
        border-radius: 15px;
        padding: 5px 15px;
    }

    .details .color-model .cm-son .cm-for.bind-color {
        border: 1px solid red;
        color: red;
        background-color: #fff4f6;
    }

    /* 删除线 */
    .details .color-model .cm-son .cm-for.disabled {
        text-decoration: line-through;
        color: #c9c7cc;
        background-color: rgba(176, 172, 172, 0.1);
    }

    /* 评论 */
    .details .comment-box {
        width: 720rpx;
        margin: 20px auto;
        background-color: white;
        border-radius: 15px;
        padding-top: 20px;
        overflow: hidden;
    }

    .details .comment-box .cb {
        font-size: 12px;
        padding: 0 8px 0;
        margin-bottom: 15px;
        display: flex;
    }

    /* 加载更多 */
    .details .comment-box .cb .cb-r {
        margin-left: auto;
        color: #C8C7CC;
    }

    .details .comment-box .cb .cb-r span {
        margin-left: 5px;
        color: #0a0a0b;
    }

    .details .comment-box .com-box {
        padding: 0 8px 0;
        margin-bottom: 15px;
    }

    .details .comment-box .com-box .hpn {
        display: flex;
        margin-bottom: 10px;
    }

    /* 评论头像 */
    .details .comment-box .com-box .hpn .head-portrait {
        height: 60rpx;
        width: 60rpx;
        margin-right: 10px;
    }

    .details .comment-box .com-box .hpn .head-portrait image {
        height: 60rpx;
        width: 60rpx;
        border-radius: 50%;
    }

    /* 昵称 */
    .details .comment-box .com-box .hpn .nick-name {
        font-size: 13px;
        margin-bottom: 4px;
    }

    /* 几星评价 */
    .details .comment-box .com-box .hpn .star {
        width: 80px;
        height: 14px;
        background-image: url(../../static/star.png);
        background-repeat: no-repeat;
    }

    /* 评论内容 */
    .details .comment-box .com-box .content {
        font-size: 14px;
        text-align: justify;
        margin-bottom: 10px;
    }

    /* 评论图片 */
    .details .comment-box .com-box .pics {
        display: flex;
        width: 750rpx;
        height: 80rpx;
    }

    .details .comment-box .com-box .pics image {
        margin-right: 10px;
        width: 80rpx;
        height: 80rpx;
    }

    /* 详情 */
    .details .shop-particulars {
        width: 720rpx;
        margin: 20px auto;
        background-color: white;
        border-radius: 15px;
        padding-top: 20px;
        overflow: hidden;
    }

    .details .shop-particulars .minutiae {
        font-size: 12px;
        padding: 0 8px 0;
        margin-bottom: 15px;
    }

    .details .shop-particulars .sp-pics {
        width: 720rpx;
    }

    .details .shop-particulars image {
        width: 720rpx;
    }

    /* 推荐 */
    .details .recommend {
        width: 750rpx;
        background-color: rgb(218, 216, 216, .3);
        padding-top: 20px;
    }

    .details .recommend .rec-w {
        font-size: 12px;
        padding: 15rpx 8px;
        margin-bottom: 15px;
    }
    /* 推荐产品布局 */
    .details .recommend .goods-flax{
        width: 720rpx;
        margin: 20px auto;
        display: flex;
        flex-wrap: wrap;
        justify-content:space-between;
    }
    .details .recommend .goods-flax .goods{
        width: 350rpx;
        background-color: white;
        margin-top: 20px;
        border-radius: 15px;
    }
    .details .recommend .goods-flax .goods image{
        border-radius: 15px 15px 0 0;
        width: 350rpx;
        height: 350rpx;
    }
</style>
