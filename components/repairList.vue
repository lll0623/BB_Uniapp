<template>
    <div class="repairList" style="padding-bottom: 60px;">
        <van-sticky><van-search :value="value" placeholder="请输入搜索关键词" show-action @search="onSearch" @cancel="onCancel" /></van-sticky>
        <van-tabs animated :active="active" swipeable="true" @change="onChange" color="#1989fa">
            <van-tab :title="'未完成(' + tab1 + ')'">
                <div class="itemUl" v-for="item in list" :key="item"  @click="goDetail(item)">
                    <div class="title">
                        <div class="fl">
                            <span class="bold">[紧急1]</span>
                            <span class="bold">2020-03-11 12:00</span>
                        </div>
                        <div class="fr">WXD-00011</div>
                        <div class="clearfix"></div>
                    </div>
                    <div class="content">
                        <div class="fl img"><van-image fit="fill" src="https://img.yzcdn.cn/vant/cat.jpeg" /></div>
                        <div class="fl pl10 text C3">
                            <p class="lineEllips1">
                                所属班组：
                                <span>强电组</span>
                            </p>
                            <p class="lineEllips2">
                                报修地点：
                                <span>绿地科技到广场A座8单元2614</span>
                            </p>
                            <p class="lineEllips2">
                                报修内容：
                                <span>绿地科技到广场A座8单元2614绿地科技到广场A座8单元2614绿地科技到广场A座8单元2614</span>
                            </p>
                        </div>
                        <div class="clearfix"></div>
                    </div>
                </div>
                <div class="C6 tc p20" v-show="finished">已全部加载</div>
            </van-tab>
            <van-tab title="待评价" :info="tab2">
               <!-- <div class="itemUl" v-for="item in list" :key="item">
                    <div class="title">
                        <div class="fl">
                            <span class="bold">[紧急]</span>
                            <span class="bold">2020-03-11 12:00</span>
                        </div>
                        <div class="fr">WXD-00011</div>
                        <div class="clearfix"></div>
                    </div>
                    <div class="content">
                        <div class="fl img"><van-image fit="fill" src="https://img.yzcdn.cn/vant/cat.jpeg" /></div>
                        <div class="fl pl10 text C3">
                            <p class="lineEllips1">
                                所属班组：
                                <span>强电组</span>
                            </p>
                            <p class="lineEllips2">
                                报修地点：
                                <span>绿地科技到广场A座8单元2614</span>
                            </p>
                            <p class="lineEllips2">
                                报修内容：
                                <span>绿地科技到广场A座8单元2614绿地科技到广场A座8单元2614绿地科技到广场A座8单元2614</span>
                            </p>
                        </div>
                        <div class="clearfix"></div>
                    </div>
                </div> -->
            </van-tab>
            <van-tab :title="'已评价(' + tab3 + ')'">
                <div class="itemUl" v-for="item in list" :key="item">
                    <!-- <div class="title">
                        <div class="fl">
                            <span class="bold">[紧急]</span>
                            <span class="bold">2020-03-11 12:00</span>
                        </div>
                        <div class="fr">WXD-00011</div>
                        <div class="clearfix"></div>
                    </div>
                    <div class="content">
                        <div class="fl img"><van-image fit="fill" src="https://img.yzcdn.cn/vant/cat.jpeg" /></div>
                        <div class="fl pl10 text C3">
                            <p class="lineEllips1">
                                所属班组：
                                <span>强电组</span>
                            </p>
                            <p class="lineEllips2">
                                报修地点：
                                <span>绿地科技到广场A座8单元2614</span>
                            </p>
                            <p class="lineEllips2">
                                报修内容：
                                <span>绿地科技到广场A座8单元2614绿地科技到广场A座8单元2614绿地科技到广场A座8单元2614</span>
                            </p>
                        </div>
                        <div class="clearfix"></div>
                    </div> -->
                </div>
            </van-tab>
        </van-tabs>
        <van-toast id="van-toast"  />
        <div class="addRepair" @click="gotoRepairAdd">
            <van-icon name="plus" class="Cf rel fs38" />
        </div>
    </div>
</template>
<script>
import Toast from '../static/vant/toast/toast';
export default {
    props: ['list1'],
    data() {
        return {
            windowHeight: '',
            list: [],
            refreshing: false,
            finished: false,
            value: '',
            tab1: 10,
            tab2: 10,
            tab3: 10,
            active: 0
        };
    },
    mounted() {
        wx.getSystemInfo({
            success: res => {
                let clientHeight = res.windowHeight;
                this.windowHeight = clientHeight;
            }
        });
        this.loadList();
    },
    methods: {
        loadList() {
            if (!this.finished) {
                Toast.loading({
                    duration: 0,
                    message: '疯狂加载中...'
                });
                setTimeout(() => {
                    if (this.refreshing) {
                        this.list = [];
                        this.refreshing = false;
                    }
                    for (let i = 0; i < 5; i++) {
                        this.list.push(this.list.length + 1);
                    }
                    this.loading = false;
                    if (this.list.length >= 20) {
                        this.refreshing = true;
                        this.finished = true;
                    }
                    wx.stopPullDownRefresh(); //	1秒后结束下拉事件
                    Toast.clear();
                }, 2000);
            }
        },
        onSearch() {
            console.log('搜索');
        },
        onCancel() {
            console.log('取消');
        },
        onChange() {
            console.log('2353456');
        },
        goDetail(id) {
            let url = '../repairDetail/main?id=' + id;
            uni.navigateTo({ url });
        },
        gotoRepairAdd() {
            let url = '../repairAdd/main';
            wx.navigateTo({ url });
        }
    },
    onPullDownRefresh() {
        this.finished = false;
        this.refreshing = true;
        console.log('下拉111');
    },
    onReachBottom() {
        this.loadList();
        console.log('已到达设置距离底部的距离');
    }
};
</script>
