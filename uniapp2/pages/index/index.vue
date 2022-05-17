<template>
    <div>
        <div class="s-input">
            <div class="i-fdj"></div>
            <div class="input" @click="goSearch()">{{text}}</div>
            <div class="i-photo"></div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                text:"裙子"
            }
        },
        methods: {
            goSearch() {
                wx.navigateTo({
                    url:"../search/search",
                })
            }
        },
    }
</script>

<style scoped>
    .s-input {
        width: 720rpx;
        height: 30px;
        margin: 0 auto;
        padding-top: 5px;
        flex: 1;
        position: relative;
    }
    .s-input .i-fdj{
        height: 15px;
        width: 15px;
        background-image: url(../../static/icon-fdj.png);
        background-repeat: no-repeat;
        background-size: 100% 100%;
        position: absolute;
        left: 15px;
        top: 14px;
    }
    .s-input .input {
        height: 30px;
        padding: 0 15px 0 45px;
        line-height: 30px;
        border-radius: 30px;
        border: 1px solid #787878;
        color: #999999;
    }

    .s-input .i-photo {
        height: 20px;
        width: 20px;
        background-image: url(../../static/icon-photo.png);
        background-repeat: no-repeat;
        background-size: 100% 100%;
        position: absolute;
        right: 15px;
        top: 10px;
    }
</style>
