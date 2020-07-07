<template>
    <div class="bindStaff">
        <div v-show="step1">
            <van-sticky>
                <van-search
                    :value="value"
                    placeholder="请输入手机号码、姓名、身份证、员工编号进行查找"
                    use-action-slot
                    @search="onSearch"
                    @change="onChangeSearch"
                    background="#1989fa"
                >
                    <view slot="action" @tap="onSearch" style="color: #FFFFFF;">搜索</view>
                </van-search>
            </van-sticky>
            <div class="staffList oneList">
                <van-radio-group :value="radio" @change="onChange">
                    <van-cell-group>
                        <van-cell v-for="(item, index) in WorkerList" :key="index" clickable :data-name="index" @click="onClick(item, index)">
                            <view slot="title">
                                <view class="van-cell-text">
                                    <van-radio class="fl" slot="right-icon" :name="index" />
                                </view>
                            </view>
                            <view>
                                <view class="fl ml20">
                                    <span class="C3">{{ item.Name }}</span>
                                    <span class="ml20 C3">{{ item.PhoneNum|phoneHide }}</span>
                                    <p class="C6">{{ item.OrganizeName }}/{{ item.DepartmentName }}</p>
                                </view>
                            </view>
                        </van-cell>
                        <div v-if="WorkerList.length == 0" class="tc p30 red">
                            暂无员工
                        </div>
                    </van-cell-group>
                </van-radio-group>
            </div>
        </div>
        <div v-show="step2">
            <div class="staffList">
                <van-cell icon="user-o">
                    <view slot="title">
                        <view class="van-cell-text">
                            <view class="fl ml20">
                                <span class="C3">{{ WorkerListJson.Name }}</span>
                                <span class="ml20 C3">{{ WorkerListJson.PhoneNum|phoneHide }}</span>
                                <p class="C6">{{ WorkerListJson.OrganizeName }}/{{ WorkerListJson.DepartmentName }}</p>
                            </view>
                        </view>
                    </view>
                </van-cell>
                <van-cell icon="" title-width="150rpx">
                    <view slot="title" style="line-height: 65rpx;">补全号码</view>
                    <div class="phone">
                        <div class="fl mr10">{{ phone1 }}</div>
                        <input type="number" maxlength="1" v-model="phone4" @input="focues(4)" :focus="focusState4" />
                        <input type="number" maxlength="1" v-model="phone5" @input="focues(5)" :focus="focusState5" />
                        <input type="number" maxlength="1" v-model="phone6" @input="focues(6)" :focus="focusState6" />
                        <input type="number" maxlength="1" v-model="phone7" @input="focues(7)" :focus="focusState7" />
                        <div class="fl ml10">{{ phone8 }}</div>
                    </div>
                </van-cell>
                <van-field :value="verifyCode" @change="onChangeBtn" left clearable label="验证码" placeholder="请输入短信验证码" border="true" use-button-slot>
                    <van-button slot="button" :disabled="disabled" size="small" @click="sendcode" type="primary">{{ getBtnTxt }}</van-button>
                </van-field>
            </div>
        </div>
        <div class="foot">
            <div v-show="step1"><van-button type="info" block @click="relation">关联</van-button></div>
            <div v-show="step2" class="tc">
                <van-button type="default" @click="before" class="fl" style="width: 35%;">上一步</van-button>
                <van-button type="info" block @click="submit" :disabled="disabledSub" class="fl" style="width: 60%;">提交验证</van-button>
                <div class="clearfix"></div>
            </div>
            <van-divider contentPosition="center" class="C3">提示</van-divider>
            <view class="fs24" style="color: #969799;line-height: 20px;">手机号码为预留管理系统中员工号码，如遇到无法匹配的情况，请联系管理员。</view>
        </div>
        <van-toast id="van-toast" />
        <van-dialog id="van-dialog" />
    </div>
</template>
<script>
import Toast from '../../static/vant/toast/toast';
import Dialog from '../../static/vant/dialog/dialog';
import { PreLogin, TryGetWorkerLoginByPreToken, SearchWorker, GetBPoneVaildateCode, BindWorkerToPubAcc } from '../../utils/http';
export default {
    data() {
        return {
            value: '',
            WorkerList: [],
            WorkerListJson: {},
            step1: true,
            step2: false,
            radio: 0,
            verifyCode: '', // 验证码
            time: 0,
            getBtnTxt: '获取验证码',
            preToken: '',
            disabled: false,
            disabledSub: true,
            phone1: '',
            phone4: '',
            phone5: '',
            phone6: '',
            phone7: '',
            phone8: '',
            focusState4: true,
            focusState5: false,
            focusState6: false,
            focusState7: false
        };
    },
    methods: {
        onChangeSearch(e) {
            // 搜索框值改变事件
            this.value = e.mp.detail;
        },
        onSearch() { // 搜索框搜索事件
            let regPhoneNum = /^(13[0-9]|14[5-9]|15[012356789]|166|17[0-8]|18[0-9]|19[8-9])[0-9]{8}$/;
            if(this.value == '') {
                Toast.fail('请输入关键字搜索！');
                return;
            };
            if(this.value.length == 11 && !regPhoneNum.test(this.value)) {
                Toast.fail("请输入正确的手机号！");
                return;
            }
            if(this.value.length == 18 && !this.checkCardIdFunc(this.value)) {
                Toast.fail("请输入正确的身份证号！");
                return;
            }
            let parm = { searchText: this.value };
            SearchWorker(parm).then(res => {
                if(res.Status && !!res.Data.WorkerList && !!res.Data.WorkerList.length > 0) {
                    this.WorkerList = res.Data.WorkerList;
                    this.WorkerListJson = res.Data.WorkerList[0];
                } else {
                    this.WorkerList = [];
                }
            });
        },
        onChange(e) {
            // 员工选择改变
            this.radio = e.mp.detail;
        },
        onClick(e, index) {
            // 员工选择改变
            this.WorkerListJson = e;
            this.radio = index;
        },
        relation() {
            if(!this.WorkerListJson.Id) {
                Toast.fail('暂无可关联员工！');
                return;
            }
            if(!this.WorkerListJson.PhoneNum) {
                Toast.fail('当前员工没有绑定手机号，请选择其它员工！');
                return;
            }
            if(!this.WorkerListJson.PhoneNum.length == 11) {
                Toast.fail("当前员工绑定的手机号无效，请选择其它员工！");
                return;
            }
            // 关联按钮
            this.step1 = false;
            this.step2 = true;
            this.phone1 = this.WorkerListJson.PhoneNum.substr(0, 3);
            this.phone4 = '';
            this.phone5 = '';
            this.phone6 = '';
            this.phone7 = '';
            this.phone8 = this.WorkerListJson.PhoneNum.substr(7, 10);
            this.focues(4);
        },
        before() {
            this.step1 = true;
            this.step2 = false;
        },
        sendcode() {
            let phone = Number(this.phone1 + this.phone4 + this.phone5 + this.phone6 + this.phone7 + this.phone8);
            let wId = this.WorkerListJson.Id;
        	let that = this;
            let reg = 11 && /^(13[0-9]|14[5-9]|15[012356789]|166|17[0-8]|18[0-9]|19[8-9])[0-9]{8}$/;
            if (phone == '') {
        		Toast.fail({
        		    message: '请输入手机号码',
        		    position: 'top',
        		    duration: 3000
        		});
            } else if (!reg.test(phone)) {
        		Toast.fail({
        		    message: '手机格式不正确',
        		    position: 'top',
        		    duration: 3000
        		});
            } else {
                let params = {
                    tel: phone,
                    wId: wId
                };
        		this.disabled = true;
                GetBPoneVaildateCode(params).then((response) => {
                    var errorText = response.Message;
                    switch (response.Status) {
                    case 200:
                        if(!!response.Data) {
                            Toast.success({
                                message: '短信验证码发送成功，请注意查收！',
                                position: 'top',
                                duration: 3000
                            });
                            this.time = 60;
                            this.disabled = true;
                            this.timer();
                            this.disabledSub = false;
                        } else {
                            Toast.fail({
                                message: '短信验证码发送失败！',
                                position: 'top',
                                duration: 3000
                            });
                            this.time = 0;
                            this.getBtnTxt = "获取验证码";
                            this.disabled = false;
                        }
                        break;
                    case 500:
                        Toast.fail({
                            message: errorText,
                            position: 'top',
                            duration: 3000
                        });
                        this.time = 0;
                        this.getBtnTxt = "获取验证码";
                        this.disabled = false;
                        break;
                    default:
                        Toast.fail({
                            message: '短信验证码发送失败！',
                            position: 'top',
                            duration: 3000
                        });
                        this.time = 0;
                        this.getBtnTxt = "获取验证码";
                        this.disabled = false;
                    }
                });
            }
        },
        timer() {
            if (this.time > 0) {
                this.time--;
                this.getBtnTxt = this.time + "s后获取";
                setTimeout(this.timer, 1000);
            } else {
                this.time = 0;
                this.getBtnTxt = "获取验证码";
                this.disabled = false;
            }
        },
        onChangeBtn(e) {
            this.verifyCode = e.mp.detail;
        },
        submit() { // 提交验证
            if(this.phone4 == "" || this.phone5 == "" || this.phone6 == "" || this.phone7 == "") {
                Toast.fail({
                    message: '请先补全号码！',
                    position: 'top',
                    duration: 3000
                });
                return;
            }
            if(this.verifyCode == "") {
                Toast.fail({
                    message: '请输入验证码！',
                    position: 'top',
                    duration: 3000
                });
                return;
            }
            let params = {
                wId: this.WorkerListJson.Id,
                preToken: this.globalData.windowData.preToken,
                verifyCode: this.verifyCode
            };
            BindWorkerToPubAcc(params).then(res => {
                if(res.Status == 200) {
                    Toast.success({
                        message: res.Message,
                        position: 'top',
                        duration: 3000
                    });
                    this.globalData.useInfo = res.Data.WorkerData;
                    this.globalData.windowData.Token = res.Data.Token;
                    this.globalData.windowData.UserId = res.Data.UserId;
                    this.globalData.windowData.CmpCode = res.Data.WorkerData.CmpCode;
                    this.globalData.windowData.isBind = true; // 绑定之后重新加载
                    let url = '../index/main';
                    uni.redirectTo({ url });
                } else {
                    Toast.fail({
                        message: res.Message,
                        position: 'top',
                        duration: 3000
                    });
                }
            });
        },
        focues(num) {
            switch (num) {
            case 4:
                if ((this.phone4 + '').length > 0) {
                    this.focusState4 = false;
                    this.focusState5 = true;
                    this.focusState6 = false;
                    this.focusState7 = false;
                } else {
                    this.focusState4 = true;
                    this.focusState6 = false;
                    this.focusState7 = false;
                }
                break;
            case 5:
                if ((this.phone5 + '').length > 0) {
                    this.focusState4 = false;
                    this.focusState5 = false;
                    this.focusState6 = true;
                    this.focusState7 = false;
                } else {
                    this.focusState4 = false;
                    this.focusState5 = true;
                    this.focusState7 = false;
                }
                break;
            case 6:
                if ((this.phone6 + '').length > 0) {
                    this.focusState4 = false;
                    this.focusState5 = false;
                    this.focusState6 = false;
                    this.focusState7 = true;
                } else {
                    this.focusState4 = false;
                    this.focusState5 = false;
                    this.focusState6 = true;
                }
                break;
            case 7:
                this.focusState4 = false;
                this.focusState5 = false;
                this.focusState6 = false;
                this.focusState7 = true;
                break;
            default:
            }
        },
        checkCardIdFunc(value) {
            let reg = /^(^[1-9]\d{7}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}$)|(^[1-9]\d{5}[1-9]\d{3}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])((\d{4})|\d{3}[Xx])$)$/;
            let city = {
                11: "北京",
                12: "天津",
                13: "河北",
                14: "山西",
                15: "内蒙古",
                21: "辽宁",
                22: "吉林",
                23: "黑龙江 ",
                31: "上海",
                32: "江苏",
                33: "浙江",
                34: "安徽",
                35: "福建",
                36: "江西",
                37: "山东",
                41: "河南",
                42: "湖北 ",
                43: "湖南",
                44: "广东",
                45: "广西",
                46: "海南",
                50: "重庆",
                51: "四川",
                52: "贵州",
                53: "云南",
                54: "西藏 ",
                61: "陕西",
                62: "甘肃",
                63: "青海",
                64: "宁夏",
                65: "新疆",
                71: "台湾",
                81: "香港",
                82: "澳门",
                91: "国外 "
            };
            if (reg.test(value) === false || !city[value.substr(0, 2)]) {
                return false;
            }else{
        		return true;
        	}
        }
    },
    filters: {
    	phoneHide: function(num) {
            if(!!num) {
                return num.substring(0, 3) + '****' + num.substring(7);
            } else {
                return "";
            }
    	}
    }
};
</script>

<style lang="scss">
.bindStaff {
    .van-search {
        height: 100rpx;
    }
    .oneList{
        .van-cell__title {
            flex: 0.2;
        }
        .van-cell__value {
            flex: 1.8;
            text-align: left;
        }
    }
    .staffList {
        padding-bottom: 200px;
        .phone {
            line-height: 65rpx;
            input {
                float: left;
                border: 1px solid #ddd;
                width: 20px;
                height: 20px;
                padding: 10rpx;
                margin-right: 10rpx;
                text-align: center;
                color: #1989fa;
            }
        }
        van-field {
            .van-cell__title {
                padding-left: 12rpx;
            }
        }
    }
    .foot {
        background-color: #ffffff;
        height: 150px;
        position: fixed;
        bottom: 0;
        padding: 20px;
        width: calc(100% - 40px);
    }
}
</style>
