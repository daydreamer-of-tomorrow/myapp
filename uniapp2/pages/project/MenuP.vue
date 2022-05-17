<template>
    <div>
        <div class="menu-box">
            <scroll-view scroll-y="true" style="height: 100px;">
                <div class="menu-content">
                    <view class="ct-f" :class="{redc: nowChinese == '价格' ? nowChoosePrice == item.value : nowChooseArr.includes(item.value)}"
                        v-for="(item, index) in option" :key="index" @click="bindColorChange(item.value)">
                        <span v-show="false"
                            :class="{redc: nowChinese == '价格' ? nowChoosePrice == item.value : nowChooseArr.includes(item.value)}">✔</span>{{item.valueDesc}}
                    </view>
                </div>
            </scroll-view>


            <div class="menu-btn">
                <button @click="$emit('countermand')">取消</button>
                <button @click="colorReplacement()">重置</button>
                <button style="background-color: red;color:white;" @click="okHandler()">确定</button>
            </div>

        </div>

    </div>
</template>

<script>
    export default {
        props: ['option', 'nowChinese', 'cfArr', 'cfPrice'],
        data() {
            return {
                redC: -1,
                nowChooseArr: [...this.cfArr],
                ziIndex: 0,
                nowChoosePrice:this.cfPrice
            }
        },
        methods: {
            // 选择颜色变化
            bindColorChange(value) {
                if(this.nowChinese == '价格'){
                    if(this.nowChoosePrice == value){
                        this.nowChoosePrice = '';
                    }else{
                        this.nowChoosePrice = value;
                    }
                }else{
                    if (this.nowChooseArr.includes(value)) {
                        this.nowChooseArr = this.nowChooseArr.filter(item => item != value);
                    } else {
                        this.nowChooseArr.push(value);
                    }
                }
                
            },
            okHandler(){
                this.$emit('ok', {
                    nowChoosePrice: this.nowChoosePrice,
                    nowChooseArr: this.nowChooseArr
                })
            },
            // 重置
            colorReplacement() {
                // 重置的时候只需要让nowChooseArr = []就行了,就可以取消选择,但这样写发现一个bug，就是重置后，nowChooseArr = []了，那就会重新刷新页面，相当与什么也没选择。解决这个bug只需要让当前所在的菜单栏中的value清空，其他菜单栏中的value不清空就可以了，我用for循环遍历option得到this.option[i].value，再用filter删去nowChooseArr中与其相同的项，达到重置的目的
                for (let i = 0; i < this.option.length; i++) {
                    this.nowChooseArr = this.nowChooseArr.filter(item => item != this.option[i].value);
                }
                console.log(this.nowChooseArr);
            }
        },
        watch: {
            nowChinese() {
                this.nowChooseArr = [...this.cfArr];
                this.nowChoosePrice = this.cfPrice;
            }
        }
    }
</script>

<style scoped>
    .menu-box {
        width: 730rpx;
        padding: 10px 0;
        margin: 0 auto;
        box-sizing: border-box;
    }

    .menu-box .menu-content {
        overflow: hidden;
        position: relative;
    }

    .menu-box .menu-content .ct-f {
        height: 20px;
        line-height: 20px;
        float: left;
        font-size: 13px;
        width: 40%;
        /* border: 1px solid #2C405A; */
        margin-right: 10px;
        margin-bottom: 5px;
        /* border-radius: 20px; */
    }

    .menu-box .menu-btn {
        height: 30px;
        display: flex;
    }

    .menu-box .menu-btn button {
        height: 30px;
        line-height: 30px;
        flex: 1;
    }

    .redc {
        color: red;
        display: block;
        margin-right: 4px;
        float: left;
    }
</style>
