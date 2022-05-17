<template>
    <div>
        <view class="box">
            <!-- 返回箭头 -->
            <div class="box-return">
                <div class="bar-return" @click="returnHandler()"></div>
            </div>
            
            <!-- tab栏 -->
            <div class="box-bar">
                <!-- 横向筛选 -->
                <scroll-view class="" scroll-x="true" :enhanced="true" :show-scrollbar="false">
                    <view class="horizontal-scroll">
                        <view class="hs-ct" :class="{cur:iscur(item), runthrough:nowIndex == index}"
                            v-for="(item, index) in filters" :key="index" @click="switchOver(index)">
                            {{tabContent(item)}}
                        </view>
                    </view>
                </scroll-view>
            </div>

            <!-- 无限滚动y 动态style-->
            <scroll-view class="box-s" scroll-y="true" :style="{height:(safeHeight - 150) + 'px'}"
                @scrolltolower="loadMore()" lower-threshold="170">
                <!-- v-for循环，让数据呈现在页面上 -->
                <view class="chunk" v-for="item in goods" :key="item.SKU" @click="redirect(item.SKU)">
                    <view class=" chunk-pic">
                        <!-- 反引号，动态src -->
                        <image :src="`http://www.zkt-it.com:5050/pic/product/${item.thumbnail}?format=450w_450h_1e_0l`"
                            mode=""></image>
                    </view>
                    <view class="chunk-ct">
                        <view class="" style="font-size: 14px;">
                            {{item.pName}}
                        </view>
                        <view class="" style="color: orange;font-size: 18px;">
                            {{item.price}}元
                        </view>
                    </view>
                </view>
            </scroll-view>

            <!-- 下拉菜单 -->
            <div class="menu" v-if="menuVisible">
                <div class="menu-inner">
                    <!-- 子组件 -->
                    <MenuP :option="filters[nowIndex].values" :nowChinese="filters[nowIndex].fieldNameDesc"
                        :cfArr="cfArr" :cfPrice="cfPrice" @ok="cfArrChange($event)" @countermand="replacementHandler()">
                    </MenuP>
                </div>
                <!-- 点击menu-inner其余部分也能取消菜单 -->
                <div class="menu-residue" @click="menuVisible = false"></div>
            </div>
        </view>
    </div>
</template>

<script>
    // 加载页码
    let page = 1;
    // 节流锁
    let lock = true;

    // 引子组件
    import MenuP from './MenuP.vue';
    export default {
        onLoad(p) {
            this.keyword = p.keyword;
            // 页面数据加载
            this.loadData();
            // 筛选数据加载
            this.loadFilters();
            // 安全高
            wx.getSystemInfo({
                success: (e) => {
                    console.log(e);
                    this.safeHeight = e.safeArea.height;
                }
            })
        },
        data() {
            return {
                // 安全高
                safeHeight: 0,
                // 页面数据
                goods: [],
                // 筛选数据
                filters: [],
                // tab栏点击的那项的下标
                nowIndex: -1,
                // 传给子组件的cf数组
                cfArr: [],
                // menu的显示隐藏
                menuVisible: false,
                // 传给子组件的单选价格
                cfPrice: '',
                keyword:''
            }
        },
        methods: {
            loadData() {
                // 开菊花图
                wx.showLoading({
                    title: '加载中...'
                });
                // 拉Ajax
                wx.request({
                    url: "http://www.zkt-it.com:5050/product-search?keyword=" + this.keyword + "&page=" + page +
                        "&pagesize=10&cf=" + this.cfArr.join(',') + "," + this.cfPrice,
                    success: (data) => {
                        // console.log(data.data.goods);
                        // 把页面缀在后面
                        this.goods = [...this.goods, ...data.data.goods];
                        // 关菊花图
                        wx.hideLoading();
                        // 数据回来开启节流锁
                        lock = true;
                    }
                })
            },

            // 加载更多
            loadMore() {
                // 判断节流锁是否打开
                if (lock) {
                    // 关闭节流锁
                    lock = false;
                    // console.log(page);
                    // 页面加一
                    page++;
                    // 拉数据
                    this.loadData();
                }
            },
            // 筛选器
            loadFilters() {
                wx.request({
                    url: "http://www.zkt-it.com:5050/filter-what?thing=" + this.keyword,
                    success: (data) => {
                        console.log(data.data.filters);
                        this.filters = data.data.filters;
                    }
                })
            },
            // tab栏
            switchOver(index) {
                if (this.nowIndex == index) {
                    this.menuVisible = false;
                    this.nowIndex = -1;
                } else {
                    this.menuVisible = true;
                    this.nowIndex = index;
                }

            },
            // 确定
            cfArrChange($event) {
                console.log($event);
                this.menuVisible = false;
                this.cfArr = $event.nowChooseArr;
                this.cfPrice = $event.nowChoosePrice;
                page = 1;
                this.goods = [];
                this.nowIndex = -1;
                this.loadData();
            },
            // 取消
            replacementHandler() {
                this.menuVisible = false;
                // 解决点击取消按钮管船不消失的问题
                this.nowIndex = -1;
            },
            // tab栏的内容变化
            tabContent(item) {
                if (item.fieldNameDesc != '价格') {
                    let count = 0;
                    let arr = [];
                    for (let i = 0; i < item.values.length; i++) {
                        if (this.cfArr.includes(item.values[i].value)) {
                            count++;
                            arr.push(item.values[i].valueDesc);
                        }
                    }
                    if (count == 0) {
                        return item.fieldNameDesc;
                    } else if (count == 1) {
                        return item.fieldNameDesc + '(' + arr[0] + ')';
                    } else {
                        return item.fieldNameDesc + '(' + arr[0] + '等' + count + '个)'
                    }
                } else {
                    if (this.cfPrice) {
                        return '价格' + '(' + this.cfPrice + ')';
                    } else {
                        return '价格';
                    }
                }

            },
            // 点击确定后tab颜色变化
            iscur(item) {
                if (item.fieldNameDesc != '价格') {
                    let count = 0;
                    for (let i = 0; i < item.values.length; i++) {
                        if (this.cfArr.includes(item.values[i].value)) {
                            count++;
                        }
                    }
                    return count != 0;
                } else {
                    if (this.cfPrice) {
                        return true;
                    } else {
                        return false;
                    }
                }
            },
            redirect(sku){
                console.log(sku);
                wx.navigateTo({
                    url:"/pages/detail/detail?id=1&sku=" + sku + "&keyword=" + this.keyword,
                })
            },
            returnHandler(){
                wx.navigateBack();
            }
        },
        // 定
        components: {
            MenuP
        }
    }
</script>

<style scoped>
    
    /* 返回箭头 */
    .box .box-return{
        height: 88px;
        width: 750rpx;
        position: relative;
    }
    .box .box-return .bar-return {
        height: 20px;
        width: 20px;
        background-image: url(../../static/return.png);
        background-repeat: no-repeat;
        background-size: 100% 100%;
        position: absolute;
        z-index: 9999;
        left: 25rpx;
        top: 50px;
    }
    .box .box-s {
        margin: 0 auto;
        background-color: #FFFFFF;
    }
    .box .box-s .chunk {
        display: flex;
        padding: 15rpx;
    }

    .box .box-s .chunk .chunk-pic {
        width: 225rpx;
        height: 225rpx;
        margin-right: 15px;
    }

    .box .box-s .chunk .chunk-pic image {
        width: 225rpx;
        height: 225rpx;
        border-radius: 8px;
    }

    .box .box-bar {
        padding: 10px;
    }
    .box .box-bar .horizontal-scroll {
        height: 30px;
        padding: 6px;
        white-space: nowrap;
    }

    .box .box-bar .horizontal-scroll .hs-ct {
        background-color: #e7e7e7;
        padding: 2px 10px;
        border-radius: 40px;
        margin-right: 10px;
        font-size: 12px;
        display: inline-block;
        position: relative;
    }

    .box .box-bar .horizontal-scroll .hs-ct.cur {
        background-color: red;
        color: white;
    }

    .box .box-bar .horizontal-scroll .hs-ct.runthrough {
        border-radius: 10px 10px 0 0;
    }

    .box .box-bar .horizontal-scroll .hs-ct.runthrough::before {
        content: '';
        position: absolute;
        top: 20px;
        left: 0;
        height: 30px;
        width: 100%;
        background-color: #e7e7e7;
    }

    /* 菜单 */
    .box .menu {
        position: fixed;
        top: 130px;
        left: 0;
        z-index: 999;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, .5);
    }

    .box .menu .menu-inner {
        width: 100%;
        background-color: #e7e7e7;
        border-radius: 0 0 10px 10px;
    }

    .box .menu .menu-residue {
        width: 100%;
        height: 100%;
    }
</style>
