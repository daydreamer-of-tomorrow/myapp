<template>
    <div>
        <div class="search-box">
            <div class="box-return">
                <div class="bar-return" @click="returnHandler()"></div>
            </div>
            <div class="s-input">
                <!-- input框默认值 -->
                <div class="default" v-if="text == ''">裙子</div>
                <!-- focus="true"调起软键盘 -->
                <input :style="{color: colorInput}" type="text" v-model="text" @confirm="goProject()"  :focus="focus" />
                <div class="i-photo"></div>
            </div>
            <div class="s-text" @click="goProject()">搜索</div>
        </div>

        <!-- 历史搜索记录 -->
        <div class="s-history">搜索历史</div>
        <div class="record">
            <span @click="recordHandler(item)" class="record-for" v-for="(item,index) in historyText" :key="index">
                {{item}}
            </span>
        </div>
        
        <!-- 智能感应 -->
       <!-- <div v-for="(name,index) in name" :key="index">
            <div v-html="moHuHTML(name)"></div>
        </div> -->
    </div>
</template>

<script>
    export default {
        // 当页面展示
        onShow() {
            wx.getStorage({
                key:"keyword",
                success:data =>{
                    this.historyText = JSON.parse(data.data);
                }
            })
        },
        data() {
            return {
                // 默认填入
                text: "",
                historyText:'',
                // 默认颜色
                colorInput: '#141414',
                // name:[],
                focus:true,
            }
        },
        methods: {
            goProject() {
                // 读
                wx.getStorage({
                    key:"keyword",
                    // 第一次输入的时候进入,fail表示失败后执行的
                    fail: (data) => {
                        console.log(data);
                        console.log("还没有数组");
                        let arr = [this.text];
                        // 设置
                        wx.setStorage({
                            key:"keyword",
                            // JSON.stringify(arr)把数组转化为字符串
                            data:JSON.stringify(arr),
                        });
                    },
                    success:data =>{
                        console.log(data);
                        // JSON.parse(data.data)把字符串转化为数组
                        let arr = JSON.parse(data.data);
                        console.log(arr);
                        arr.push(this.text);
                        // 数组去重,集合Set方法
                        arr = [...new Set(arr)];
                        // arr.reduce方法
                        // arr = arr.reduce((a, b) => a.includes(b) ? a : [...a, b], []);
                        // 设置
                        wx.setStorage({
                            key:"keyword",
                            // JSON.stringify(arr)把数组转化为字符串
                            data:JSON.stringify(arr),
                        });
                    }
                });
                
                if (this.text == '') {
                    // 当text为空时,默认搜索是裙子
                    this.text = '裙子';
                }
                wx.navigateTo({
                    url: "../project/project?keyword=" + this.text,
                });
            },
            // 返回上一级页面
            returnHandler(){
                wx.navigateBack({
                    delta:1
                });
            },
            // 点击搜索历史记录搜索
            recordHandler(item){
                this.text = item;
                wx.navigateTo({
                    url: "../project/project?keyword=" + this.text,
                });
            },
            // 智能感应的字的字体变红
            // moHuHTML(name){
            //     let index = name.indexOf(this.text);
            //     return name.slice(0, index) + "<span style='color:red'>" + this.text + "</span>" + name.slice(index + this.text.length);
            // }
        },
        // watch:{
            // 智能感应
            // text(v){
            //     wx.request({
            //         url:"http://www.zkt-it.com:5050/user?kw=" + v,
            //         success:data=>{
            //             this.name = data.data.results.map(item=> item.name);
            //         }
            //     })
            // }
        // }
    }
</script>

<style scoped>
    /* 顶部搜索盒子 */
    .search-box {
        height: 120px;
        width: 750rpx;
        padding-bottom: 5px;
        display: flex;
        overflow: hidden;
        align-items: flex-end;
    }

    /* 返回箭头 */
    .box-return {
        width: 50px;
        height: 30px;
        /* background-color: #007AFF; */
        margin-right: 15px;
        position: relative;
    }

    .box-return .bar-return {
        height: 20px;
        width: 20px;
        background-image: url(../../static/return.png);
        background-repeat: no-repeat;
        background-size: 100% 100%;
        position: absolute;
        z-index: 9999;
        left: 15px;
        top: 5px;
    }

    /* 搜索框 */
    .search-box .s-input {
        height: 30px;
        flex: 1;
        position: relative;
    }
    /* 默认值 */
    .search-box .s-input .default{
        height: 20px;
        line-height: 20px;
        width: 30px;
        font-size: 14px;
        position: absolute;
        left: 15px;
        top: 5px;
        color: #808080;
    }
    .search-box .s-input input {
        height: 30px;
        line-height: 30px;
        font-size: 14px;
        padding: 0 15px;
        border-radius: 30px;
        border: 1px solid red;
    }
    .search-box .s-input .i-photo {
        height: 20px;
        width: 20px;
        background-image: url(../../static/icon-photo.png);
        background-repeat: no-repeat;
        background-size: 100% 100%;
        position: absolute;
        right: 15px;
        top: 5px;
    }

    .search-box .s-text {
        margin-left: 15px;
        width: 50px;
        height: 30px;
        line-height: 30px;
        font-size: 14px;
        /* background-color: #696978; */
    }

    .s-history {
        margin-top: 15px;
        font-size: 14px;
        padding-left: 25px;
    }
    .record{
        padding-left: 30px;
        margin-top: 10px;
    }
    .record .record-for{
        font-size: 10px;
        margin-right:10px;
        background-color: #c7c7c7;
        display: inline-block;
        padding: 3px 10px;
        border-radius: 15px;
    }
</style>
