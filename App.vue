<script>
import { PreLogin, TryGetWorkerLoginByPreToken, GetItems } from './utils/http';
import globalData from './utils/global'; // 全局参数
export default {
    onLaunch: function() {
        // console.log('App Launch');
        if (wx.canIUse('getUpdateManager')) {
            const updateManager = wx.getUpdateManager()
            updateManager.onCheckForUpdate(function (res) {
                console.log('onCheckForUpdate====', res)
                // 请求完新版本信息的回调
                if (res.hasUpdate) {
                    console.log('res.hasUpdate====')
                    updateManager.onUpdateReady(function () {
                        wx.showModal({
                            title: '更新提示',
                            content: '新版本已经准备好，是否重启应用？',
                            success: function (res) {
                                // res: {errMsg: "showModal: ok", cancel: false, confirm: true}
                                if (res.confirm) {
                                    // 新的版本已经下载好，调用 applyUpdate 应用新版本并重启
                                    updateManager.applyUpdate()
                                }
                            }
                        })
                    })
                    updateManager.onUpdateFailed(function () {
                        // 新的版本下载失败
                        wx.showModal({
                            title: '已经有新版本了哟~',
                            content: '新版本已经上线啦~，请您删除当前小程序，重新搜索打开哟~'
                        })
                    })
                }
            })
        }
    },
    onShow: function() {
        // console.log('App Show');
        let params = {
            keys: [
                'Ls.CS.EvaluateType', // 维修评价类型
            ]
        }
        GetItems(params).then(res => {
            if (res.Status == 200) {
                globalData.windowData.GetItemsData = res.Data;
            }
            console.log(globalData);
        })
    },
    onHide: function() {
        // console.log('App Hide');
    }
};
</script>

<style lang="scss">
	/* 这部分相当于原生小程序的 app.wxss */
	.container {
	    background-color: white;
	    font-size: 14px;
	}
	.lineThrough {
	    text-decoration: line-through;
	}
	.ellips {
	    overflow: hidden;
	    text-overflow: ellipsis;
	    white-space: nowrap;
	    word-wrap: normal;
	}
	/* 超出一行就显示... */
	.lineEllips1 {
	    -webkit-line-clamp: 1;
	    display: -webkit-box;
	    -webkit-box-orient: vertical;
	    overflow: hidden;
	}
	/* 超出两行就显示... */
	.lineEllips2 {
	    -webkit-line-clamp: 2;
	    display: -webkit-box;
	    -webkit-box-orient: vertical;
	    overflow: hidden;
	}
	.clearfix:after {
	    content: '';
	    display: block;
	    height: 0;
	    font-size: 0;
	    clear: both;
	    visibility: hidden;
	}
	textarea {
	    line-height: initial;
	}
	.fl {
	    float: left;
	}
	.fr {
	    float: right;
	}
	.tc {
	    text-align: center;
	}
	.tr {
	    text-align: right;
	}
	.tl {
	    text-align: left;
	}
	.abs {
	    position: absolute;
	}
	.rel {
	    position: relative;
	}
	.fix {
	    position: fixed;
	}
	.bold {
	    font-weight: bold;
	}
	.block {
	    display: block;
	}
	.inlineB {
	    display: inline-block;
	}
	.marAuto {
	    margin-left: auto;
	    margin-right: auto;
	}
	.CZhu {
	    color: #1989fa;
	}
	.Cf {
	    color: #fff;
	}
	.C0 {
	    color: #000;
	}
	.C3 {
	    color: #333;
	}
	.C6 {
	    color: #666;
	}
	.C9 {
	    color: #999;
	}
	.Cf4 {
	    color: #f4f4f4;
	}
	.Ce {
	    color: #eee;
	}
	.red {
	    color: #eb3223;
	}
	.yellow {
	    color: #f5c520;
	}
	.green {
	    color: #0da697;
	}
	.bgF {
	    background: #fff;
	}
	.bgF4 {
	    background: #f4f4f4;
	}
	.bgY {
	    background: #f5c520;
	}
	.bgY0 {
	    background: #fdf6ec;
	}
	.fs22 {
	    font-size: 22rpx;
	}
	.fs24 {
	    font-size: 24rpx;
	}
	.fs26 {
	    font-size: 26rpx;
	}
	.fs28 {
	    font-size: 28rpx;
	}
	.fs30 {
	    font-size: 30rpx;
	}
	.fs32 {
	    font-size: 32rpx;
	}
	.fs34 {
	    font-size: 34rpx;
	}
	.fs36 {
	    font-size: 36rpx;
	}
	.fs38 {
	    font-size: 38rpx;
	}
    .ft600{
        font-weight: 600;
    }
	
	/*padding*/
	.pl10 {
	    padding-left: 10rpx;
	}
	.pl30 {
	    padding-left: 30rpx;
	}
	.p20 {
	    padding: 20rpx;
	}
	.p30 {
	    padding: 30rpx;
	}
	.pr20 {
	    padding-right: 20rpx;
	}
    .pt20{
        padding-top: 20rpx;
    }
	.pr40 {
	    padding-right: 40rpx;
	}
	.p040 {
	    padding: 0 40rpx;
	}
	.pb30 {
	    padding-bottom: 30rpx;
	}
	.pb40 {
	    padding-bottom: 40rpx;
	}
	/*margin*/
	.mt5 {
	    margin-top: 5rpx;
	}
	.mt10 {
	    margin-top: 10rpx;
	}
	.mt20 {
	    margin-top: 20rpx;
	}
	.mt30 {
	    margin-top: 30rpx;
	}
	.mt40 {
	    margin-top: 40rpx;
	}
	.mt45 {
	    margin-top: 45rpx;
	}
	.mt60 {
	    margin-top: 60rpx;
	}
	.mt80 {
	    margin-top: 80rpx;
	}
	.mb5 {
	    margin-bottom: 5rpx;
	}
	.mb10 {
	    margin-bottom: 10rpx;
	}
	.mb15 {
	    margin-bottom: 15rpx;
	}
	.mb20 {
	    margin-bottom: 20rpx;
	}
	.mb30 {
	    margin-bottom: 30rpx;
	}
	.mb40 {
	    margin-bottom: 40rpx;
	}
	.mr10 {
	    margin-right: 10rpx;
	}
	.mr20 {
	    margin-right: 20rpx;
	}
	.mr25 {
	    margin-right: 25rpx;
	}
	.mr30 {
	    margin-right: 30rpx;
	}
	.ml10 {
	    margin-left: 10rpx;
	}
	.ml20 {
	    margin-left: 20rpx;
	}
	.line18 {
	    line-height: 36rpx;
	}
	.bordBeee {
	    border-bottom: 1px solid #eee;
	}
	/*width*/
	.w50 {
	    width: 50% !important;
	}
	.w100 {
	    width: 100% !important;
	}
	.row-tit {
	    width: 140rpx;
	    left: 0;
	    top: 0;
	    min-height: 88rpx;
	    line-height: 88rpx;
	}
	.row-val {
	    margin-left: 175rpx;
	    min-height: 88rpx;
	    line-height: 88rpx;
	}
	div.imp:after {
	    content: '*';
	    position: absolute;
	    font-size: 16px;
	    color: red;
	    right: -10px;
	    top: 2px;
	}
    .van-toast__text {
        text-align: center;
    }
	/* 附件样式 */
	.van-uploader__upload--disabled {
	    display: none !important;
	}
	/* 时间轴样式 */
	.verticalTimeline {
	    height: 100%;
	    overflow: hidden;
	    overflow-y: auto;
	    line-height: 30px;
	    overflow: -moz-scrollbars-none;
	    padding-bottom: 180px;
	    .timeline {
	        position: relative;
	        padding: 20px 0 20px;
	        list-style: none;
	        li {
	            position: relative;
	            margin-bottom: 20px;
	            display: inline-block;
	            width: 100%;
	            .timelineBadge {
	                z-index: 80;
	                position: absolute;
	                top: 16px;
	                left: 80px;
	                width: 25px;
	                height: 25px;
	                border-radius: 50% 50% 50% 50%;
	                text-align: center;
	                font-size: 1em;
	                line-height: 25px;
	                color: #1989fa;
	                border: 3px solid #1989fa;
	                background-color: white;
	            }
	            .timelineBadge.danger {
	                border: 3px solid #ee0a24 !important;
	                color: #ee0a24 !important;
	            }
	            .timelineBadge.success {
	                color: #07c160 !important;
	                border: 3px solid #07c160 !important;
	            }
	            .timelinePanel {
	                float: left;
	                position: relative;
	                width: 88px;
	                padding: 0px;
	            }
	            .timelineTitle {
	                position: relative;
	                width: calc(100% - 108px);
	                padding: 0px 5px 0 5px;
	                float: right;
	            }
	        }
	        li:before {
	            content: ' ';
	            display: inherit !important;
	            position: absolute;
	            top: 36px;
	            bottom: 0;
	            left: 95px;
	            width: 2px;
	            margin-left: -1.5px;
	            background-color: #eeeeee;
	            margin-bottom: -50px;
	        }
	        li:last-child:before {
	            content: ' ';
	            display: inherit !important;
	            position: absolute;
	            top: 36px;
	            bottom: 0;
	            left: 95px;
	            width: 2px;
	            margin-left: -1.5px;
	            background-color: #eeeeee;
	            height: 0px;
	        }
	        li:first-child:before {
	            height: auto;
	        }
	        li:before,
	        li:after {
	            content: ' ';
	            display: table;
	        }
	    }
	}
</style>
