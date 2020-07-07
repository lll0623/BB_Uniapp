<template>
    <div class="container">
        <div class="repairList" v-show="activeNav == 0" style="padding-bottom: 50px;">
            <van-sticky><van-search :value="SearchKey" placeholder="请输入搜索关键词" show-action @search="onSearch" @change="onChangeSearch" @cancel="onCancel" /></van-sticky>
            <van-tabs animated :active="active" swipeable="true" @change="onChange" color="#1989fa">
                <van-tab :title="'未完成(' + tab1 + ')'" class="rel" v-show="active == 0">
                    <div v-if="list.length == 0 && params.isShow" class="red tc p20">暂无数据</div>
                    <div class="itemUl" v-for="(item, index) in list" :key="index" v-show="active == 0" @click="goDetail(item)">
                        <div class="title">
                            <div class="fl">
                                <span v-if="item.Level == 1" class="bold">[普通]</span>
                                <span v-if="item.Level == 2" class="bold">[紧急]</span>
                                <span class="bold">{{ item.Date }}</span>
                            </div>
                            <div class="fr">{{ item.Code }}</div>
                            <div class="clearfix"></div>
                        </div>
                        <div class="content">
                            <div class="fl img"><van-image fit="fill" :src="defaultImg" /></div>
                            <div class="fl pl10 text C3">
                                <p class="lineEllips1">
                                    所属班组：
                                    <span>{{ item.TeamName }}</span>
                                </p>
                                <p class="lineEllips2">
                                    报修地点：
                                    <span>{{ item.RepairObjectName }}</span>
                                </p>
                                <p class="lineEllips2">
                                    报修内容：
                                    <span>{{ item.RepairContent }}</span>
                                </p>
                            </div>
                            <div class="clearfix"></div>
                        </div>
                        <div class="C6 tc p20" v-if="list.length - 1 == index" v-show="finished">已全部加载</div>
                    </div>
                </van-tab>
                <van-tab title="待评价" :info="tab2" v-show="active == 1">
                    <div v-if="list1.length == 0 && params1.isShow" class="red tc p20">暂无数据</div>
                    <div class="itemUl" v-for="(item, index) in list1" v-show="active == 1" :key="index" @click="goDetail(item)">
                        <div class="title">
                            <div class="fl">
                                <span v-if="item.Level == 1" class="bold">[普通]</span>
                                <span v-if="item.Level == 2" class="bold">[紧急]</span>
                                <span class="bold">{{ item.Date }}</span>
                            </div>
                            <div class="fr">{{ item.Code }}</div>
                            <div class="clearfix"></div>
                        </div>
                        <div class="content">
                            <div class="fl img"><van-image fit="fill" :src="defaultImg" /></div>
                            <div class="fl pl10 text C3">
                                <p class="lineEllips1">
                                    所属班组：
                                    <span>{{ item.TeamName }}</span>
                                </p>
                                <p class="lineEllips2">
                                    报修地点：
                                    <span>{{ item.RepairObjectName }}</span>
                                </p>
                                <p class="lineEllips2">
                                    报修内容：
                                    <span>{{ item.RepairContent }}</span>
                                </p>
                            </div>
                            <div class="clearfix"></div>
                        </div>
                        <div class="C6 tc p20" v-if="list1.length - 1 == index" v-show="finished1">已全部加载</div>
                    </div>
                </van-tab>
                <van-tab :title="'已评价(' + tab3 + ')'" v-show="active == 2">
                    <div v-if="list2.length == 0 && params2.isShow" class="red tc p20">暂无数据</div>
                    <div class="itemUl" v-for="(item, index) in list2" :key="index" v-show="active == 2" @click="goDetail(item)">
                        <div class="title">
                            <div class="fl">
                                <span v-if="item.Level == 1" class="bold">[普通]</span>
                                <span v-if="item.Level == 2" class="bold">[紧急]</span>
                                <span class="bold">{{ item.Date }}</span>
                            </div>
                            <div class="fr">{{ item.Code }}</div>
                            <div class="clearfix"></div>
                        </div>
                        <div class="content">
                            <div class="fl img"><van-image fit="fill" :src="defaultImg" /></div>
                            <div class="fl pl10 text C3">
                                <p class="lineEllips1">
                                    所属班组：
                                    <span>{{ item.TeamName }}</span>
                                </p>
                                <p class="lineEllips2">
                                    报修地点：
                                    <span>{{ item.RepairObjectName }}</span>
                                </p>
                                <p class="lineEllips2">
                                    报修内容：
                                    <span>{{ item.RepairContent }}</span>
                                </p>
                            </div>
                            <div class="clearfix"></div>
                        </div>
                        <div class="C6 tc p20" v-if="list2.length - 1 == index" v-show="finished2">已全部加载</div>
                    </div>
                </van-tab>
            </van-tabs>
            <van-toast id="van-toast" />
            <div class="addRepair" @click="gotoRepairAdd"><van-icon name="plus" class="Cf rel fs38" /></div>
        </div>
        <div class="mine rel" v-show="activeNav == 2" :style="{ height: windowHeight }">
            <div class="mineTital">
                <div class="leftImg fl"><van-image round class="img" :src="mineData.HeaderPic" /></div>
                <div class="fl pt20">
                    <p class="ft600 fs34">{{ mineData.WorkerName }}</p>
                    <p class="fs28">{{ mineData.OrgName }}</p>
                    <p class="fs28">{{ mineData.DepName }}</p>
                </div>
                <div style="clear:both;"></div>
            </div>
            <div class="mineList">
                <van-cell title="我的消息" icon="chat-o" use-label-slot="true" is-link>
                    <view><van-tag type="danger" round>new</van-tag></view>
                </van-cell>
                <van-cell title="系统通知" icon="volume-o" is-link></van-cell>
                <!-- <txv-video vid="a0935dmbzaf" playerid="txv1"></txv-video> -->
            </div>
            <div class="bottomBtn"><van-button type="danger" @click="UnBind" block>解绑</van-button></div>
        </div>
        <van-toast id="van-toast" />
        <van-tabbar :active="activeNav" @change="onChangeNav">
            <van-tabbar-item icon="wap-home">工作</van-tabbar-item>
            <van-tabbar-item class="zidingyi">
                <van-uploader @afterread="afterRead" multiple><van-icon name="photograph" /></van-uploader>
            </van-tabbar-item>
            <van-tabbar-item icon="manager">我的</van-tabbar-item>
        </van-tabbar>
        <van-toast id="van-toast" />
        <van-dialog id="van-dialog" />
    </div>
</template>
<script>
import { PreLogin, TryGetWorkerLoginByPreToken, RepairApplet, UnbindAccRela } from '../../utils/http';
import defaultImg from '../../static/images/default.png';
import Toast from '../../static/vant/toast/toast';
import Dialog from '../../static/vant/dialog/dialog';
export default {
    onShow() {
        // 评价回来刷新列表页面
        if (this.globalData.windowData.indexTab == 0 || this.globalData.windowData.indexTab == 1 || this.globalData.windowData.indexTab == 2) {
            // this.active = this.globalData.windowData.indexTab;
            this.Refresh();
        }
        // 添加维修管理回来刷新页面
        if (this.globalData.windowData.indexReload) {
            this.Refresh();
        }
        this.globalData.windowData.indexReload = false;
        this.globalData.windowData.indexTab = null;
    },
    data() {
        return {
            defaultImg,
            mineData: {
                HeaderPic: '',
                WorkerName: '',
                OrgName: '',
                DepName: ''
            },
            windowHeight: '',
            params: {
                PageIndex: 1,
                PageSize: 15,
                isShow: false
            },
            params1: {
                PageIndex: 1,
                PageSize: 15,
                isShow: false
            },
            params2: {
                PageIndex: 1,
                PageSize: 15,
                isShow: false
            },
            list: [],
            list1: [],
            list2: [],
            finished: false,
            finished1: false,
            finished2: false,
            SearchKey: '',
            tab1: 0,
            tab2: 0,
            tab3: 0,
            active: 0,
            activeNav: 0,
            mine: {}
        };
    },
    mounted() {
        wx.getSystemInfo({
            success: res => {
                let clientHeight = res.windowHeight;
                this.windowHeight = clientHeight;
            }
        });
        this.WeiXinFun();
    },
    methods: {
        WeiXinFun() {
            let that = this;
            wx.login({
                success(res) {
                    if (res.code) {
                        Toast.loading({
                            duration: 0,
                            mask: true,
                            message: '授权登录中...'
                        });
                        PreLogin({ code: res.code }).then(res => {
                            Toast.clear();
                            if (res.Status == 200 && !!res.Data) {
                                that.globalData.windowData.preToken = res.Data;
                                TryGetWorkerLoginByPreToken({ preToken: res.Data }).then(ret => {
                                    if (!ret.Data || !ret.Data.WorkerData) {
                                        // 没有查询到绑定员工
                                        Dialog.alert({
                                            title: '授权成功！',
                                            message: '当前没有绑定的账号信息。请先绑定员工！'
                                        }).then(() => {
                                            let url = '../bindStaff/main';
                                            uni.redirectTo({ url });
                                        });
                                    } else {
                                        // 有查询到绑定员工
                                        Toast.success({
                                            message: '登陆成功',
                                            position: 'top',
                                            duration: 3000
                                        });
                                        that.globalData.useInfo = ret.Data.WorkerData;
                                        that.globalData.windowData.Token = ret.Data.Token;
                                        that.globalData.windowData.UserId = ret.Data.UserId;
                                        that.globalData.windowData.CmpCode = ret.Data.WorkerData.CmpCode;
                                        that.loadList(0);
                                        that.loadList(1);
                                        that.loadList(2);
                                    }
                                });
                            } else {
                                Toast.fail(res.Message);
                            }
                        });
                    }
                }
            });
        },
        onChangeSearch(e) {
            // 搜索框值改变事件
            this.SearchKey = e.mp.detail;
        },
        onSearch() {
            if (this.active == 0) this.params.PageIndex = 1;
            this.list = [];
            this.finished = false;
            if (this.active == 1) this.params1.PageIndex = 1;
            this.list1 = [];
            this.finished1 = false;
            if (this.active == 2) this.params2.PageIndex = 1;
            this.list2 = [];
            this.finished2 = false;
            this.loadList(this.active);
        },
        loadList(num) {
            if (num == 0) {
                if (!this.finished) {
                    Toast.loading({
                        duration: 0,
                        message: '疯狂加载中...'
                    });
                    let params = {
                        IsASC: false,
                        FieId: 'Code',
                        PageIndex: this.params.PageIndex,
                        PageSize: 15,
                        SearchKey: this.SearchKey,
                        State: '1, 2, 3', // 状态 in
                        IsCheck: '' // 待回访 false 已回访 true
                    };
                    RepairApplet({ model: params }).then(res => {
                        if (res.Status == 200) {
                            let list = res.Data.Data;
                            res.Data.RowCount > 99 ? (this.tab1 = 99) : (this.tab1 = res.Data.RowCount);
                            if (list.length == this.params.PageSize) {
                                this.params.PageIndex += 1;
                            } else {
                                this.finished = true;
                            }
                            this.list = this.list.concat(list);
                            Toast.clear();
                        } else {
                            Toast.fail({
                                message: res.Message,
                                position: 'top',
                                duration: 3000
                            });
                            Toast.clear();
                        }
                        this.params.isShow = true;
                    });
                }
            }
            if (num == 1) {
                if (!this.finished1) {
                    Toast.loading({
                        duration: 0,
                        message: '疯狂加载中...'
                    });
                    let params = {
                        IsASC: false,
                        FieId: 'Code',
                        PageIndex: this.params1.PageIndex,
                        PageSize: 15,
                        SearchKey: this.SearchKey,
                        State: '', // 状态 in
                        IsCheck: false // 待回访 false 已回访 true
                    };
                    RepairApplet({ model: params }).then(res => {
                        if (res.Status == 200) {
                            let list = res.Data.Data;
                            res.Data.RowCount > 99 ? (this.tab2 = 99) : (this.tab2 = res.Data.RowCount);
                            if (list.length == this.params1.PageSize) {
                                this.params1.PageIndex += 1;
                            } else {
                                this.finished1 = true;
                            }
                            this.list1 = this.list1.concat(list);
                            Toast.clear();
                        } else {
                            Toast.fail({
                                message: res.Message,
                                position: 'top',
                                duration: 3000
                            });
                            Toast.clear();
                        }
                        this.params1.isShow = true;
                    });
                }
            }
            if (num == 2) {
                if (!this.finished2) {
                    Toast.loading({
                        duration: 0,
                        message: '疯狂加载中...'
                    });
                    let params = {
                        IsASC: false,
                        FieId: 'Code',
                        PageIndex: this.params2.PageIndex,
                        PageSize: 15,
                        SearchKey: this.SearchKey,
                        State: '', // 状态 in
                        IsCheck: 'true' // 待回访 false 已回访 true
                    };
                    RepairApplet({ model: params }).then(res => {
                        if (res.Status == 200) {
                            let list = res.Data.Data;
                            res.Data.RowCount > 99 ? (this.tab3 = 99) : (this.tab3 = res.Data.RowCount);
                            if (list.length == this.params2.PageSize) {
                                this.params2.PageIndex += 1;
                            } else {
                                this.finished2 = true;
                            }
                            this.list2 = this.list2.concat(list);
                            Toast.clear();
                        } else {
                            Toast.fail({
                                message: res.Message,
                                position: 'top',
                                duration: 3000
                            });
                            Toast.clear();
                        }
                        this.params2.isShow = true;
                    });
                }
            }
            Toast.clear();
            wx.stopPullDownRefresh();
        },
        goDetail(item) {
            let url = '../repairDetail/main?id=' + item.Id;
            uni.navigateTo({ url });
        },
        gotoRepairAdd() {
            let url = '../repairAdd/main?num=' + this.active;
            wx.navigateTo({ url });
        },
        onChangeNav(e) {
            let index = e.detail;
            if (index === 0 || index === 2) {
                this.activeNav = index;
                if (index === 0) {
                    wx.setNavigationBarTitle({
                        title: '维修管理'
                    });
                } else {
                    wx.setNavigationBarTitle({
                        title: '我的'
                    });
                    this.mineData.HeaderPic = this.globalData.windowData.ImgUrl + this.globalData.useInfo.HeaderPic;
                    this.mineData.WorkerName = this.globalData.useInfo.WorkerName;
                    this.mineData.OrgName = this.globalData.useInfo.OrgName;
                    this.mineData.DepName = this.globalData.useInfo.DepName;
                }
            }
        },
        onChange(e) {
            this.active = e.detail.index;
            wx.pageScrollTo({
                scrollTop: 0
            });
            if (this.active == 0 && this.list.length == 0) {
                this.loadList(this.active);
            }
            if (this.active == 1 && this.list1.length == 0) {
                this.loadList(this.active);
            }
            if (this.active == 2 && this.list2.length == 0) {
                this.loadList(this.active);
            }
        },
        afterRead(e) {
            let fileList = JSON.stringify(e);
            let url = '../repairAdd/main?fileList=' + fileList;
            uni.navigateTo({ url });
        },
        UnBind() {
            let params = {
                refId: this.globalData.useInfo.WorkerId,
                refType: '3'
            };
            Dialog.confirm({
                title: '确定解绑当前员工？',
                message: '解绑后需要回到绑定员工页面重新绑定。'
            })
                .then(() => {
                    UnbindAccRela(params).then(res => {
                        if (res.Status == 200) {
                            uni.showToast({
                                title: '解绑成功',
                                duration: 2000
                            });
                            this.globalData.useInfo = null;
                            setTimeout(function() {
                                let url = '../bindStaff/main';
                                uni.redirectTo({ url });
                            }, 2000);
                        } else {
                            Toast.fail({
                                message: res.Message,
                                position: 'top',
                                duration: 3000
                            });
                        }
                    });
                })
                .catch(() => {});
        },
        Refresh() {
            this.finished = false;
            this.finished1 = false;
            this.finished2 = false;
            this.refreshing = true;
            this.params.PageIndex = 1;
            this.params1.PageIndex = 1;
            this.params2.PageIndex = 1;
            this.list = [];
            this.list1 = [];
            this.list2 = [];
            this.loadList(0);
            this.loadList(1);
            this.loadList(2);
        }
    },
    onPullDownRefresh() {
        // 下拉刷新
        this.Refresh();
    },
    onReachBottom() {
        this.loadList(this.active);
    }
};
</script>

<style lang="scss">
.container {
    background-color: white;
}
.message {
    color: red;
    padding: 10px;
    text-align: center;
}
.zidingyi {
    van-icon {
        color: $uni-color-primary;
        font-size: 45px;
    }
}
// 维修列表页面
.repairList {
    background-color: white;
    overflow-y: auto;
    .van-search {
        height: 100rpx;
    }
    van-tabs {
        position: relative;
        padding-bottom: 80px;
        van-tab {
            overflow-y: auto;
        }
        .van-tabs__wrap {
            position: fixed !important;
            width: 100%;
            height: 100rpx;
            z-index: 99;
        }
        .van-tab__pane {
            margin-top: 110rpx;
        }
        .itemUl {
            padding: 0 10rpx;
            border-bottom: 3px solid #eeeeee;
            background-color: white;
            z-index: 88;
            position: relative;
            &:last-child {
                border-bottom: 0px;
            }
            .p20 {
                border-top: 3px solid #eeeeee;
            }
            .title {
                padding: 10rpx;
                border-bottom: 1px solid #eeeeee;
            }
            .content {
                padding: 10rpx;
                .img {
                    width: 35%;
                    height: 200rpx;
                    .van-image {
                        height: 100%;
                        width: 100%;
                    }
                }
                .text {
                    width: calc(65% - 10rpx);
                    font-size: 32rpx;
                }
            }
        }
    }
    .addRepair {
        position: fixed;
        background-color: rgba(25, 137, 250, 0.7215686274509804);
        z-index: 99;
        bottom: 80px;
        right: 20px;
        width: 50px;
        height: 50px;
        border-radius: 50px;
        text-align: center;
        van-icon {
            top: 17px;
            z-index: 1000;
            font-weight: 700;
        }
    }
}
.mine {
    background-color: #eeeeee;
    overflow-y: auto;
    .mineTital {
        background-color: #1989fa;
        color: white;
        padding: 3%;
        display: inline-block;
        width: 94%;
        min-height: 100rpx;
        margin-bottom: 20rpx;
        .leftImg {
            width: 200rpx;
            height: 150rpx;
            .van-image {
                width: 150rpx;
                height: 150rpx;
            }
        }
    }
    .bottomBtn {
        position: fixed;
        bottom: 80px;
        padding: 20px;
        width: calc(100% - 40px);
    }
}
</style>
