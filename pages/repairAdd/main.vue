<template>
    <div class="repairAdd">
        <div class="mineTital">
            <div class="leftImg fl">
                <van-image round class="img" :src="HeadPic" />
            </div>
            <div class="fl pt20">
                <p class="ft600 fs34">{{ Name }}</p>
                <p class="fs28">{{ Originaise }}</p>
                <p class="fs28">{{ PStructName }}</p>
            </div>
            <div style="clear:both;"></div>
        </div>
        <div class="contentFiled">
            <van-cell icon="" use-label-slot="true" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text">报修区域</view>
                </view>
                <view>
                    <van-radio-group
                    	:value="RepairObjectType"
                        @change="radioChange"
                    	class="van-radio van-radio--horizontal">
                        <van-radio class="fl" name="1">公共区域</van-radio>
                        <van-radio class="fl" name="2" style="margin-left: 1rem;">业户区域</van-radio>
                    </van-radio-group>
                </view>
            </van-cell>
            <van-cell icon="" is-link="" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text rel">报修地址</view>
                </view>
                <view class="rel">
                    <input class="tr" placeholder="点击选择地址" type="text"
                        :value="RepairObjectName"
                        @click="showRoomList = true; shuKuang = false;"
                        disabled>
                </view>
            </van-cell>
            <van-cell icon="" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text rel">联系人</view>
                </view>
                <view class="rel">
                    <input class="tr" :value="RepairLinkMan" placeholder="默认当前员工" type="text">
                </view>
            </van-cell>
            <van-cell icon="" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text rel">联系电话</view>
                </view>
                <view class="rel">
                    <input class="tr" :value="RepairLinkPhone" maxlength="11" placeholder="默认当前员工的号码" type="number">
                </view>
            </van-cell>
            <van-cell icon="" is-link="" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text rel">选择班组</view>
                </view>
                <view class="rel">
                    <input class="tr" placeholder="点击选择班组" type="text"
                        :value="teamJson.text"
                        @click="clickTeamList(teamJson.index)"
                        disabled>
                </view>
                <van-popup
                	:show="showTeamList"
                    @click-overlay="showTeamList = false; shuKuang = true;"
                	position="bottom">
                    <van-picker 
                		show-toolbar
                		:default-index="teamJson.index"
                		:loading="loadingTeam"
                		:columns="teamData"
                		@confirm="onConfirmTeamFn"
                		@cancel="showTeamList = false;shuKuang = true;" />
                </van-popup>
            </van-cell>
            <van-cell icon="" use-label-slot="true" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text">紧急程度</view>
                </view>
                <view>
                    <van-radio-group
                    	:value="Level"
                        @change="radioChange2"
                    	class="van-radio van-radio--horizontal">
                        <van-radio class="fl" name="1">普通维修</van-radio>
                        <van-radio class="fl" name="2" style="margin-left: 1rem;">紧急维修</van-radio>
                    </van-radio-group>
                </view>
            </van-cell>
            <van-cell icon="" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text rel">报修内容</view>
                </view>
                <view class="rel">
                    <textarea placeholder='请输入介绍信息' v-model="RepairContent" class="tl" v-show="shuKuang"
                        :minlength="min" :maxlength="max" @input="inputs">
                    </textarea>
                    <text class="currentWordNumber">{{currentWordNumber}}/{{max}}</text>
                </view>
            </van-cell>
            <van-cell icon="" required title-width="150rpx">
                <view slot="title">
                    <view class="van-cell-text rel">报修图片</view>
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
        </div>
        <div class="bottomBtn">
            <van-button type="info" :loading="disabled" @click="SubmitRepairFun" :loading-text="loadingSubText"  block>提交</van-button>
        </div>
        <van-toast id="van-toast" />
        <van-popup class="search" style="width:100%;height:100%;"
        	:show="showRoomList"
            @click-overlay="showRoomList = false; shuKuang = true;"
            close-on-click-overlay
        	position="right">
            <form action="/" target="frameFile">
            	<van-search
            		:value="filterValue"
            		background="#1989fa"
                    use-action-slot
            		placeholder="请输入搜索关键词"
                    @search="onSearch"
                    @change="onChangeSearch"
            	>
                    <view slot="action" @tap="onSearch" style="color: #FFFFFF;">搜索</view>
                </van-search>
                <iframe name='frameFile' style="display: none;"></iframe>
            </form>
            <div class="el-breadcrumb">
                <van-icon name="wap-home" @click="QueryPropertyList()"/>
                <span>{{ breadcrumbs }}</span>
            </div>
            <div class="minePStruct">
                <ul class="listItem">
                    <div v-for="(item,index) in options" :key="index" >
                        <li v-if="!!item.PStructName && !!item.PStructId" @click="CheckPS(item.PStructId,item.PStructType,item.PStructName, index)" :class="{'active': activeLi == index ? true : false, 'rel': true, 'ellips': true}">
                            {{ item.PStructName }}
                            <van-icon class="abs" name="arrow" />
                        </li>
                    </div>
                </ul>
            </div>
            <div class="footButton">
            	<van-button class="close" @click="showRoomList=false; shuKuang = true;">关闭</van-button>
            	<van-button class="save" type="info" @click="saveFj()">确定</van-button>
            </div>
        </van-popup>
    </div>
</template>
<script>
import { SubmitRepair, GetRoomListByCode, GetTeamInfo, queryPropertyList, stringSort } from '../../utils/http.js';
import Toast from '../../static/vant/toast/toast';
let formName = {
    RepairObjectName: '报修地址',
    RepairLinkMan: '联系人',
    RepairLinkPhone: '联系电话',
    TeamId: '班组',
    RepairContent: '报修内容',
    Files: '报修图片'
};
export default {
    data() {
        return {
            activeLi: -1,
            loadingSubText: '提交中...',
            shuKuang: true,
            formName,
            HeadPic: this.globalData.windowData.ImgUrl + this.globalData.useInfo.HeaderPic,
            Name: this.globalData.useInfo.WorkerName,
            RepairLinkMan: this.globalData.useInfo.WorkerName,
            Originaise: this.globalData.useInfo.OrgName,
            PStructName: this.globalData.useInfo.DepName,
            RepairLinkPhone: this.globalData.useInfo.WPhone,
            min: 5,
            max: 200,
            currentWordNumber: 0,
            RepairContent: '',
            RepairObjectType: '1',
            Level: '1',
            options: [], // 房产数组数据
            RepairObjectName: '',
            filterValue: '', // 房产搜索值
            breadcrumbs: "全部", // 面包屑
            PStructType: 1,
            RepairObjectId: "",
            showRoomList: false, // 报修地址picker
            teamData: [], // 班组数据
            teamJson: { // 班组选中的数据
                text: '',
                value: '',
                index: '0'
            },
            loadingTeam: true, // 班组pick的loading
            showTeamList: false, // 班组地址picker
            fileList: [],
            Files: [],
            disabled: false
        };
    },
    mounted() {
        if(this.$root.$mp.query.fileList) {
            let fifileList = JSON.parse(this.$root.$mp.query.fileList);
            this.afterRead (fifileList);
        }
        // this.GetRoomListByCodeFun(); // 获取房产数据
        this.GetTeamInfoFun(); // 获取班组数据
        this.QueryPropertyList();
    },
    methods: {
        onChangeSearch(e) {
            // 搜索框值改变事件
            this.filterValue = e.mp.detail;
        },
        onSearch() { // 搜索框搜索事件
            if(this.filterValue != "") {
                let parm = {};
                // console.log(this.filterValue)
                if(this.PStructType == 1) {
                    parm = {
                        type: this.PStructType,
                        name: this.filterValue
                    };
                } else {
                    parm = {
                        type: this.PStructType,
                        name: this.filterValue,
                        parentId: this.RepairObjectId
                    };
                }
                queryPropertyList(parm).then(res => {
                    if(res.Status == 200) {
                        this.options = stringSort(res.Data, "PStructName", true);
                    }else {
                        Toast.fail({
                            message: res.Message,
                            position: 'top',
                            duration: 3000
                        });
                    }
                });
            }else {
                this.QueryPropertyList();
            }
        },
        QueryPropertyList() {
            this.PStructType = 1;
            this.RepairObjectId = "";
            this.RepairObjectName = '';
            let parm = {
                type: 1
            };
            queryPropertyList(parm).then(res => {
                if(res.Status == 200) {
                    this.options = stringSort(res.Data, "PStructName", true);
                    this.breadcrumbs = "全部";
                }else {
                    Toast.fail({
                        message: res.Message,
                        position: 'top',
                        duration: 3000
                    });
                }
            });
        },
        CheckPS(parentId, type, name, index) {
            this.activeLi = index;
            this.RepairObjectId = parentId;
            this.breadcrumbs = name; // 面包屑赋值
            this.RepairObjectName = name;
            type = Number(type) +1;
            // if(this.RepairObjectType != 1) {
            if(type < 6) {
                let parm = {
                    type: type,
                    parentId: parentId
                };
                queryPropertyList(parm).then(res =>{
                    if(res.Status == 200) {
                        this.options = stringSort(res.Data, "PStructName", true);
                    }else {
                        Toast.fail({
                            message: res.Message,
                            position: 'top',
                            duration: 3000
                        });
                    }
                });
            }
            // } else {
            //     if(type < 6) {
            //         let parm = {
            //             type: type,
            //             parentId: parentId
            //         };
            //         queryPropertyList(parm).then(res =>{
            //             if(res.Status == 200) {
            //                 this.options = stringSort(res.Data, "PStructName", true);
            //             }else {
            //                 Toast.fail({
            //                     message: res.Message,
            //                     position: 'top',
            //                     duration: 3000
            //                 });
            //             }
            //         });
            //     }
            // }
            this.PStructType = type;
        },
        saveFj() {
            if(this.PStructType != 6 && this.RepairObjectType == '2') {
                Toast.fail({
                    message: "请选择房间！",
                    position: 'top',
                    duration: 2000
                });
                this.RepairObjectId = '';
                this.RepairObjectName = '';
                return;
            }
            console.log(this.RepairObjectId)
            this.showRoomList = false;
            this.shuKuang = true;
        },
        GetTeamInfoFun() {
            let params = {
                "IsASC": false,
                "FieId": "Code",
                "PageIndex": 1,
                "PageSize": 10000,
                PropertyId: this.RepairObjectId,
                IsReported: 0
            };
            GetTeamInfo(params).then(res => {
                console.log(res);
                if (res.Status == 200) {
                    this.teamData = [];
                    for (var i = 0, item; (item = res.Data.Data[i++]);) {
                        this.teamData.push({
                            text: item.Name,
                            value: item.Id
                        });
                    }
                } else {
                    Toast.fail(res.Message);
                }
                this.loadingTeam = false;
            });
        },
        inputs (e) {
            let len = parseInt(this.RepairContent.length);
            if (len > this.max) return;
            this.currentWordNumber = len;
            if (this.currentWordNumber === 200) {
                Toast('报修内容超出字数限制！');
            }
        },
        radioChange (e) {
            console.log(e);
            this.RepairObjectType = e.mp.detail;
            // 重置报修地址数据
            this.breadcrumbs = '';
            this.RepairObjectId = '';
            this.RepairObjectName = '';
            this.QueryPropertyList();
        },
        radioChange2 (e) {
            console.log(e);
            this.Level = e.mp.detail;
        },
        clickTeamList (index) { // 下拉选择点击
            this.GetTeamInfoFun();
            console.log(index);
            this.teamJson.index = index;
            this.showTeamList = true;
            this.shuKuang = false;
        },
        onConfirmTeamFn (value, index) {
            console.log(value);
            this.teamJson = value.detail.value;
            this.teamJson.index = value.detail.index;
            this.showTeamList = false;
            this.shuKuang = true;
        },
        afterRead (e) {
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
                            console.log(res);
                            // 上传完成需要更新 fileList
                            that.fileList.push({url: that.globalData.windowData.ImgUrl + JSON.parse(res.data).Data});
                            that.Files.push({Url: JSON.parse(res.data).Data, Type: 1 });
                        }
                    });
                });
            }
        },
        deleteFile (e) {
            this.fileList.splice(e.mp.detail.index, 1);
            this.Files.splice(e.mp.detail.index, 1);
        },
        SubmitRepairFun() {
            let params = {
                DepartmentId: this.globalData.useInfo.DepId,
                DepartmentName: this.globalData.useInfo.DepName,
                Level: this.Level,
                RepairContent: this.RepairContent,
                RepairLinkMan: this.RepairLinkMan,
                RepairLinkPhone: this.RepairLinkPhone,
                RepairObjectId: this.RepairObjectId,
                RepairObjectName: this.RepairObjectName,
                RepairObjectType: this.RepairObjectType,
                RepairType: 1, // 1-电话 2- 3-
                SourceId: '',
                SourceType: 4, // 0-默认 1-联系单转单 2- App 3- 返工 20-其它 4- 微信小程序
                TeamId: this.teamJson.value,
                Files: this.Files
            };
            let isSum = true;
            for (let item in this.formName) {
                if (params[item] == "") {
                    Toast.fail(this.formName[item] + '不能为空');
                    isSum = false;
                    return;
                }
            };
            if (isSum) {
                this.disabled = true;
                SubmitRepair(params).then(res => {
                    if (res.Status == 200) {
                        Toast.success(res.Message);
                        this.globalData.windowData.indexReload = true;
                        setTimeout(()=>{
                            this.disabled = false;
                            wx.navigateBack({
                                delta: 1
                            });
                        }, 2000);
                    } else {
                        this.disabled = false;
                        Toast.fail(res.Message);
                    }
                });
            }
        }
    }
};
</script>

<style lang="scss">
    .repairAdd{
        background-color: white;
        .mineTital {
            background-color: #1989fa;
            color: white;
            padding: 3%;
            display: inline-block;
            width: 94%;
            min-height: 1rem;
            border-bottom: 0.3rem solid #EEEEEE;
            .leftImg {
                width: 200rpx;
                height: 150rpx;
                .van-image {
                    width: 150rpx;
                    height: 150rpx;
                }
            }
        }
        .contentFiled{
            padding: 0 0 90px 0;
            textarea {
                height: 80px !important;
                padding: 10rpx 10rpx 40rpx 10rpx;
                border-radius: 10rpx;
                width: calc(100% - 20rpx);
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
            z-index: 99;
        }
        .search{
            .van-popup{
                height: 100%;
                width: 100%;
            }
            .footButton{
            	position: fixed !important;
            	bottom: 0;
            	width: 100%;
            	background-color: #f7f8fa;
            	padding: 4%;
            }
            van-button.save button {
                width: 50%;
                color: white;
            	margin-left: 10%;
            }
            van-button.close button {
                width: 30%;
            }
            .minePStruct{
                height: calc(100% - 200px);
                overflow-y: auto;
                .listItem {
                    color: #333;
                    li{
                        background-color: white;
                        padding: 8px 5px;
                        border-bottom: 1px solid #dedede;
                        color: #333;
                        padding-right: 15px;
                        .abs{
                            top: 10px;
                            right: 5px;
                        }
                    }
                    li:active{
                        background-color: #e6e6e6;
                    }
                    li.active{
                        background-color: #1989fa;
                        color: white;
                    }
                }
            }
            .el-breadcrumb {
                padding: 10px 5px 10px 5px;
                border-bottom: 1px solid #ccc;
                .van-icon {
                    margin-right: 10px;
                    font-size: 20px;
                }
                label {
                    margin: 0 3px;
                }
            }
        }
    }
</style>
