<template>
    <div class="repairDetail">
        <div class="contentFiled">
            <van-sticky>
                <van-cell title="报修地址" icon="" use-label-slot="true" title-width="150rpx">
                    <view class="tl">
                        {{ viewJson.RepairObjectName }}
                    </view>
                </van-cell>
                <van-cell title="维修内容" icon="" use-label-slot="true" title-width="150rpx">
                    <view class="tl">
                        {{ viewJson.RepairContent }}
                    </view>
                </van-cell>
                <div class="block bgF">
                    <van-uploader :file-list="fileList" :disabled="true" :deletable="false" />
                </div>
            </van-sticky>
            <van-row class="mb30 block">
                <div class="clearfix verticalTimeline">
                    <ul class="timeline">
                        <li v-for="(item, index) in timeList" :key="index">
                            <div class="timelinePanel">
                                <div class="tc">
                                    <h4 class="bold">{{ item.time }}</h4>
                                    <small class="C9 fs22">{{ item.date }}</small>
                                </div>
                            </div>
                            <div v-if="item.type == '0'" class="timelineBadge success">{{ item.typeName }}</div>
                            <div v-else-if="item.type == '1'" class="timelineBadge danger">{{ item.typeName }}</div>
                            <div v-else class="timelineBadge success">{{ item.typeName }}</div>
                            <div class="timelineTitle">
                                <p class="bold pl30 fs32 C6">{{ item.name }}</p>
                                <p class="pl30 fs22 C9 line18">{{ item.content }}</p>
                                
                                <div class="w100" v-if="item.EvalList && item.EvalList.length>0 && item.typeName == '评'">
                                    <van-cell icon="" title-width="150rpx">
                                        <view slot="title">
                                            <view class="van-cell-text rel">客户满意度</view>
                                        </view>
                                        <view class="rel tl">
                                            <van-rate class="fl" readonly :value="item.CommentLevel" />
                                            <!-- <span class="fl block" v-if="item.CommentLevel == 1">不满意</span>
                                            <span class="fl block" v-if="item.CommentLevel == 2">一般</span>
                                            <span class="fl block" v-if="item.CommentLevel == 3">满意</span>
                                            <span class="fl block" v-if="item.CommentLevel == 4">很满意</span>
                                            <span class="fl block" v-if="item.CommentLevel == 5">非常满意</span> -->
                                        </view>
                                    </van-cell>
                                    <van-cell icon="" title-width="150rpx" v-for="(items, idx) in item.EvalList" :key="idx">
                                        <view slot="title">
                                            <view class="van-cell-text rel">{{ items.EvaluateTypeName }}</view>
                                        </view>
                                        <view class="rel tl">
                                            <van-rate readonly :value="items.EvaluateSatisfaction" />
                                        </view>
                                    </van-cell>
                                </div>
                                <div class="w100">
                                    <van-uploader v-if="!!item.imgList && item.imgList.length>0 " :file-list="item.imgList" :disabled="true" :deletable="false" />
                                    <view v-if="!!item.videoList && item.videoList.length>0" class="uploader_video rel fl" v-for="(items, indexs) in item.videoList" :key="indexs">
                                        <video :src="items.url" 
                                            show-fullscreen-btn="true"
                                            class="video" enable-play-gesture="true" controls="controls">
                                        </video>
                                    </view>
                                </div>
                            </div>
                        </li>
                    </ul>
                </div>
            </van-row>
        </div>
        <!-- <div class="bottomBtn" v-show=""> -->
        <div class="bottomBtn" v-show="showAppraise">
            <van-button type="info" block @click="appraise">评价</van-button>
        </div>
        <van-toast id="van-toast" />
    </div>
</template>
<script>
import { GetRepairBilID, CaoQuery, GetAPIAttachFileVM} from '../../utils/http.js';
import Toast from '../../static/vant/toast/toast';
export default {
    data() {
        return {
            Id: '',
            showAppraise: false,
            viewJson: {
                RepairObjectName: '',
                RepairContent: ''
            },
            fileList: [],
            timeList: []
        };
    },
    mounted() {
        this.Id = this.$root.$mp.query.id; // 单据id
        this.GetRepairBilIDFun(); // 详情信息
        this.CaoQueryFun(); // 时间轴
    },
    methods: {
        GetRepairBilIDFun() {
            let params = {
                Id: this.Id
            };
            Toast.loading({
                duration: 0,
                message: '疯狂加载中...'
            });
            GetRepairBilID(params).then(res => {
                if(res.Status == 200) {
                    this.viewJson = res.Data;
                    let AttachFileList = res.Data.AttachFileList;
                    if(!!AttachFileList && AttachFileList.length > 0) {
                        for(let i=0; i < AttachFileList.length; i++) {
                            let item = AttachFileList[i];
                            this.fileList.push({
                                url: this.globalData.windowData.ImgUrl + item.FileUrl
                            });
                        }
                    }
                    if(!res.Data.CheckManId && res.Data.State == 6) {
                        this.showAppraise = true;
                    }
                } else {
                    Toast.fail({
                        message: res.Message,
                        position: 'top',
                        duration: 3000
                    });
                }
                Toast.clear();
            });
        },
        CaoQueryFun() {
            let params = {
                "PageIndex": 1,
                "PageSize": 999,
                "BillId": this.Id, // 单据id
            };
            CaoQuery({ model: params }).then(res => {
                if(res.Status == 200) {
                    if(!!res.Data.Data && res.Data.Data.length > 0) {
                        let jsonData = res.Data.Data;
                        jsonData.sort(function(a, b) {
                            return Date.parse(a.CreateDate)-Date.parse(b.CreateDate);
                        });
                        for(let i=0; i < jsonData.length; i++) {
                            let item = jsonData[i];
                            let fileImg = [], fileVideo = [];
                            for(let m = 0; m < item.FileList.length; m++) {
                                if(item.FileList[m].IsVideo) {
                                    fileVideo.push({
                                        url: this.globalData.windowData.ImgUrl + item.FileList[m].FileUrl
                                    });
                                } else {
                                    fileImg.push({
                                        url: this.globalData.windowData.ImgUrl + item.FileList[m].FileUrl
                                    });
                                }
                            }
                            this.timeList.push({
                                time: item.CreateDate.substr(11, 5),
                                date: item.CreateDate.substr(0, 10).replace(/\//g, "-"),
                                type: item.Type,
                                CommentLevel: item.CommentLevel,
                                typeName: item.Title.substr(0, 1),
                                name: item.Title,
                                content: item.Content,
                                imgList: fileImg,
                                videoList: fileVideo,
                                EvalList: item.EvalList
                            });
                            // this.GetAPIAttachFileVMFun(item.Id);
                        }
                    }
                } else {
                    Toast.fail({
                        message: res.Message,
                        position: 'top',
                        duration: 3000
                    });
                }
            });
        },
        GetAPIAttachFileVMFun(DataId) {
            let params = {
                "PageIndex": 1,
                "PageSize": 99,
                "BillId": this.Id, // 单据id
                "Type": 1,
                "DataId": DataId // 操作日志id
            };
            GetAPIAttachFileVM({ model: params }).then(res => {
                console.log(res);
            });
        },
        appraise() {
            let url = '../appraise/main?id=' + this.Id;
            wx.redirectTo({ url });
        }
    }
};
</script>

<style lang="scss">
    .repairDetail{
        background-color: white;
        .contentFiled{
            padding: 0 0 20px 0;
            .van-uploader{
                width: 100%;
                .van-uploader__preview{
                    width: calc(25% - 8px);
                    height: 80px;
                    margin: 0 4px 8px 4px;
                    .van-uploader__preview-image{
                        width: 100% !important;
                        height: 100% !important;
                    }
                }
            }
        }
        .bottomBtn{
            position: fixed;
            bottom: 0;
            background-color: #eee;
            padding: 20px;
            width: calc(100% - 40px);
            z-index: 999;
        }
    }
    .uploader_video {
        width: 90px;
        height: 90px;
        margin: 8px 8px 8px 0;
        .video {
            width: 100%;
            height: 100%;
        }
    }
</style>
