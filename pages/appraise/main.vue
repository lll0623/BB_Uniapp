<template>
    <div class="container">
        <div class="contentFiled">
            <van-cell icon="" use-label-slot="true" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text">是否合格</view>
                </view>
                <view>
                    <van-radio-group
                    	:value="formData.IsQualified"
                        @change="repairChange"
                    	class="van-radio van-radio--horizontal">
                        <van-radio class="fl" name="1">合格</van-radio>
                        <van-radio class="fl" name="2" style="margin-left: 1rem;">不合格</van-radio>
                    </van-radio-group>
                </view>
            </van-cell>
            <van-cell icon="" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text rel">客户满意度</view>
                </view>
                <view class="rel tl">
                    <van-rate class="fl" :value="formData.CommentLevel" @change="changeRateCom" />
                    <span class="fl block">{{ CommentLevelText }}</span>
                </view>
            </van-cell>
            <van-cell icon="" required title-width="150rpx" v-for="(item, idx) in EvaluateType" :key="idx">
                <view slot="title">
                    <view class="van-cell-text rel">{{ item.ItemName }}</view>
                </view>
                <view class="rel tl">
                    <van-rate :id="'rate-'+item.ItemValue" :name="item.ItemValue" :value="0" @change="changeRate" />
                </view>
            </van-cell>
            <van-cell icon="" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text rel">评价说明</view>
                </view>
                <view class="rel">
                    <textarea placeholder='请输入介绍信息' v-model="formData.CommentMemo"
                        :minlength="appraiseMin" :maxlength="appraiseMax" @input="appraiseFun">
                    </textarea>
                    <text class="currentWordNumber">{{appraiseNumber}}/{{appraiseMax}}</text>
                </view>
            </van-cell>
            <div v-show="isRepair">
                <van-cell icon="" use-label-slot="true" required title-width="150rpx">
                    <view slot="title">
                        <view class="van-cell-text">返工</view>
                    </view>
                    <view>
                        <van-radio-group
                        	:value="formData.IsNeedRework"
                            @change="reworkChange"
                        	class="van-radio van-radio--horizontal">
                            <van-radio class="fl mt27" name="1">需要</van-radio>
                            <van-radio class="fl mt27" name="2" style="margin-left: 1rem;">不需要</van-radio>
                        </van-radio-group>
                    </view>
                </van-cell>
                <div v-show="isRework">
                    <van-cell icon="" required title-width="150rpx">
                        <view slot="title">
                            <view class="van-cell-text rel">返工原因</view>
                        </view>
                        <view class="rel">
                            <textarea placeholder='请输入返工原因' v-model="formData.ReworkMemo"
                                :minlength="reworkReasonMin" :maxlength="reworkReasonMax" @input="reworkReasonFun">
                            </textarea>
                            <text class="currentWordNumber">{{reworkReasonNumber}}/{{reworkReasonMax}}</text>
                        </view>
                    </van-cell>
                </div>
            </div>
            <van-cell icon="" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text rel">图片附件</view>
                </view>
                <view class="rel tl">
                   （最多上传四张照片）
                </view>
                <van-row class="block tl">
                    <van-col span="24" class="rel block mt30 ">
                        <van-uploader
                            :file-list="fileList"
                            max-count="4"
                            multiple
                            @delete="deleteFile"
                            @afterread="afterRead" />
                    </van-col>
                </van-row>
            </van-cell>
            <van-cell icon="" title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text rel">上传视频</view>
                </view>
                <van-row class="block tl">
                    <view class="uni-uploader__file">
                        <view class="uploader_video rel fl" v-for="(item, index) in fileListVideo" :key="index">
                            <van-icon class="abs" name="clear"  @click="deleteFileVideo(index)"/>
                            <video :src="item.url"
                                show-fullscreen-btn="true"
                                class="video" enable-play-gesture="true" controls="controls">
                            </video>
                        </view>
                        <van-col span="24" class="rel block fl" style="margin-top: 8px;">
                            <van-uploader
                                accept="video"
                                max-duration="10"
                                @delete="deleteFileVideo"
                                @afterread="afterReadVideo" />
                        </van-col>
                    </view>
                </van-row>
            </van-cell>
        </div>
        <div class="bottomBtn">
            <van-button type="info" :loading="subLoading" @click="submit" block loading-text="提交中...">提交</van-button>
        </div>
        <van-toast id="van-toast" />
    </div>
</template>
<script>
import { ReviewTestRepair } from '../../utils/http.js';
import Toast from '../../static/vant/toast/toast';
export default {
    data() {
        return {
            EvaluateType: this.globalData.windowData.GetItemsData["Ls.CS.EvaluateType"],
            subLoading: false,
            Id: '',
            appraiseMin: 5,
            appraiseMax: 200,
            appraiseNumber: 0,
            reworkReasonMin: 5,
            reworkReasonMax: 200,
            reworkReasonNumber: 0,
            CommentLevelText: '',
            formData: {
                IsQualified: '1',
                CommentMemo: '',
                CommentLevel: 0, // 客户满意度
                IsNeedRework: '1', // bool
                ReworkMemo: ''
            },
            fileList: [],
            FileList: [],
            fileListVideo: [],
            FileListVideo: [],
            aPIEvaluateVMList: [], // 星级数组
            isRepair: false,
            isRework: true
        };
    },
    mounted() {
        this.Id = this.$root.$mp.query.id; // 单据id
        this.EvaluateType.forEach(v => {
            this.aPIEvaluateVMList.push({
                "EvaluateTypeId": v.ItemId,
                "EvaluateTypeName": v.ItemName,
                "EvaluateSatisfaction": '',
                "Sort": v.SortCode
            })
        })
    },
    methods: {
        changeRateCom(v) {
            let val = v.detail;
            if (val == 1) this.CommentLevelText = '不满意'
            if (val == 2) this.CommentLevelText = '一般'
            if (val == 3) this.CommentLevelText = '满意'
            if (val == 4) this.CommentLevelText = '很满意'
            if (val == 5) this.CommentLevelText = '非常满意'
            this.formData.CommentLevel = val;
        },
        changeRate(val) {
            console.log(val.detail)
        },
        afterReadVideo(e) {
            console.log(e);
            let that = this;
            let file = e.detail.file;
            if(file) {
                wx.uploadFile({
                    url: that.globalData.windowData.Host + '/File/Save',
                    filePath: file.path,
                    name: 'image',
                    formData: {
                        'user': 'test'
                    },
                    header: {
                        "Content-Type": "multipart/form-data",
                        'CmpCode': this.globalData.windowData.CmpCode,
                        'LsAppCode': this.globalData.windowData.LsAppCode,
                    },
                    success(res) {
                        console.log(res);
                        // 上传完成需要更新 fileList
                        that.fileListVideo.push({url: that.globalData.windowData.ImgUrl + JSON.parse(res.data).Data});
                        that.FileListVideo.push({FileUrl: JSON.parse(res.data).Data, FileName: '', IsVideo: true });
                    }
                });
            }
        },
        deleteFileVideo(e) {
            this.fileListVideo.splice(e, 1);
            this.FileListVideo.splice(e, 1);
        },
        appraiseFun(e) { // 评价说明
            let len = parseInt(this.formData.CommentMemo.length);
            if (len > this.max) return;
            this.appraiseNumber = len;
            if (this.appraiseNumber === 200) {
                Toast('评价说明超出字数限制！');
            }
        },
        reworkReasonFun(e) { // 返工原因
            let len = parseInt(this.formData.ReworkMemo.length);
            if (len > this.max) return;
            this.reworkReasonNumber = len;
            if (this.reworkReasonNumber === 200) {
                Toast('返工原因超出字数限制！');
            }
        },
        repairChange(e) {
            this.formData.IsQualified = e.mp.detail;
            (e.mp.detail === '1') ? this.isRepair = false : this.isRepair = true;
        },
        reworkChange(e) {
            this.formData.IsNeedRework = e.mp.detail;
            (e.mp.detail === '1') ? this.isRework = true : this.isRework = false;
        },
        afterRead(e) {
            let that = this;
            let file = e.detail.file;
            if(file && file.length > 0) {
                file.forEach(item => {
                    wx.uploadFile({
                        url: that.globalData.windowData.Host + '/File/Save',
                        filePath: item.path,
                        name: 'image',
                        formData: {
                            'user': 'test'
                        },
                        header: {
                            "Content-Type": "multipart/form-data",
                            'CmpCode': this.globalData.windowData.CmpCode,
                            'LsAppCode': this.globalData.windowData.LsAppCode,
                        },
                        success(res) {
                            // 上传完成需要更新 fileList
                            that.fileList.push({url: that.globalData.windowData.ImgUrl + JSON.parse(res.data).Data});
                            that.FileList.push({FileUrl: JSON.parse(res.data).Data, FileName: '', IsVideo: false });
                        }
                    });
                });
            }
        },
        deleteFile(e) {
            this.fileList.splice(e.mp.detail.index, 1);
            this.FileList.splice(e.mp.detail.index, 1);
        },
        submit() {
            let isTo = true;
            this.aPIEvaluateVMList.forEach((v) => {
                if (!isTo) { return; }
                console.log(this.selectComponent(`#rate-${v.Sort}`).data.innerValue)
                let val = this.selectComponent(`#rate-${v.Sort}`).data.innerValue;
                if (val == 0) {
                    Toast.fail(''+v.EvaluateTypeName+'未评价');
                    isTo = false;
                } else {
                    v.EvaluateSatisfaction = val;
                }
            })
            console.log(this.aPIEvaluateVMList)
            if (this.formData.CommentLevel == 0) {
                Toast.fail('客户满意度未评价');
            }
            if(this.formData.CommentMemo == "") {
                Toast.fail('评价说明不能为空');
            }
            if(this.formData.IsQualified == '1') {
                this.formData.IsNeedRework = "2";
                this.formData.ReworkMemo = "";
                this.isRework = false;
            }
            if(this.formData.IsQualified == '2' && this.formData.IsNeedRework == "1" && this.formData.ReworkMemo == "") {
                Toast.fail('返工原因不能为空');
                return;
            }
            if(this.FileList.length == 0) {
                Toast.fail('图片不能为空');
                return;
            }
            if(this.formData.IsNeedRework == "2") {
                this.formData.ReworkMemo = "";
            }
            let params = {
                BillId: this.Id,
                CheckMan: this.globalData.useInfo.WorkerName,
                CheckManId: this.globalData.useInfo.WorkerId,
                ReworkMan: this.globalData.useInfo.WorkerName,
                ReworkManId: this.globalData.useInfo.WorkerId,
                Type: 3,
                IsQualified: Number(this.formData.IsQualified),
                CommentMemo: this.formData.CommentMemo,
                CheckMemo: this.formData.CommentMemo,
                IsNeedRework: this.formData.IsNeedRework == '1' ? true : false,
                ReworkMemo: this.formData.ReworkMemo,
                FileList: this.FileList.concat(this.FileListVideo),
                CommentLevel: this.formData.CommentLevel,
                aPIEvaluateVMList: this.aPIEvaluateVMList,
                ReworkDate: new Date()
            };
            if(this.formData.IsNeedRework == "2") {
                params.ReworkMan = "";
                params.ReworkManId = "";
            }
            this.subLoading = true;
            ReviewTestRepair({ vm: params }).then(res => {
                if (res.Status == 200) {
                    Toast.success(res.Message);
                    if(this.formData.IsNeedRework == '1') {
                        this.globalData.windowData.indexTab = 0;
                    } else {
                        this.globalData.windowData.indexTab = 2;
                    }
                    setTimeout(()=>{
                        wx.navigateBack({
                            delta: 1
                        });
                    }, 2000);
                } else {
                    Toast.fail(res.Message);
                }
                this.subLoading = false;
            });
        }
    }
};
</script>
<style lang="scss">
.contentFiled{
    background-color: white;
    padding: 0 0 90px 0;
    textarea {
        height: 80px !important;
        padding: 10rpx 10rpx 40rpx 10rpx;
        width: calc(100% - 20rpx);
        text-align: left;
    }
    .currentWordNumber {
        position: absolute;
        bottom: 0rpx;
        right: 30rpx;
        color: #888;
    }
}
.bottomBtn{
    position: fixed;
    bottom: 0;
    background-color: #eee;
    padding: 20px;
    width: calc(100% - 40px);
}
.uploader_video {
    width: 90px;
    height: 90px;
    margin: 8px 8px 8px 0;
    .video {
        width: 100%;
        height: 100%;
    }
    .abs{
        top: -8px;
        right: -8px;
        z-index: 1;
        font-size: 18px;
        background-color: #fff;
        border-radius: 50%;
    }
}
</style>
