<template>
        <view class="uni-padding-wrap uni-common-mt" >
			<rich-text :nodes="vediodata.title" style="word-break:break-all;"></rich-text>
			<div style="margin: 0; padding: 0;" @tap="viewTab">
				<video  class="video" :id="vediodata.id" :ref="vediodata.id" :src="vediodata.src" :controls="true" :loop="true"
									:show-center-play-btn="false" > 
				</video> 
			</div>
			<button @tap="share(['shareAppMessage','shareTimeline'])" open-type="share">分享给好友</button>
<!--            <button v-if="!loadError" :loading="loading" :disabled="loading" type="primary" class="btn" @click="show">显示广告</button>
            <button v-if="loadError" :loading="loading" :disabled="loading" type="primary" class="btn" @click="reloadAd">重新加载广告</button> -->		
			
			<view class="uni-list">
				<block v-for="item in data" :key="item.id">
					<view class="uni-list-cell" hover-class="uni-list-cell-hover" @click="imgclick(item)">
						<view class="uni-media-list">
							<image class="uni-media-list-logo" :src="item.imgsrc"></image>
							 <view class="uni-media-list-body">
								<view class="uni-media-list-text-top">
									<rich-text  :nodes="item.title" style="word-break:break-all;"></rich-text>
								</view>
								<view class="uni-media-list-text-bottom">
									<text >播放次数：{{item.playcnt}}</text>
								</view>
							</view>
						</view>
					</view>
				</block>
			</view>
			<view class="uni-loadmore" v-if="showLoadMore">{{loadMoreText}}</view>
		</view>

</template>

<script>
	import {
		mapState,
		mapMutations,
		mapActions
	} from 'vuex'
    const ERROR_CODE = [-5001, -5002, -5003, -5004, -5005, -5006];

    export default {
        data() {
            return {
				data: [],
				vediodata:{},
                loading: false,
                loadError: false,
				loadMoreText: "加载中...",
				showLoadMore: false
            }
        },
        onReady() {
            // #ifdef APP-PLUS
            this.adOption = {
                adpid: '__UNI__230AE34'
            };
            // #endif
            // #ifdef MP-WEIXIN
            this.adOption = {
                adUnitId: ''
            };
			 this.createAd();
            // #endif
        },
		computed: {
			...mapState(['activeOpen'])
		},
		onLoad(option) {
			this.vediodata = this.activeOpen;  
			this.initData();
			//console.log("opt?="+ this.vediodata.src);
		},
		onUnload() {
			this.max = 0,
			this.data = [],
			this.loadMoreText = "加载更多",
			this.showLoadMore = false;
		},
		onReachBottom() {
			console.log("onReachBottom");
			if (this.max > 40) {
				this.loadMoreText = "没有更多数据了!"
				return;
			}
			this.showLoadMore = true;
			setTimeout(() => {
				this.setListData();
			}, 300);
		},
		onPullDownRefresh() {
			console.log('onPullDownRefresh');
			let tt=Math.floor((Math.random()*4)+1);
			this.initData(tt*10);
		},
        methods: {
			...mapMutations(['setActiveOpen']),
            createAd() {
                var rewardedVideoAd = this.rewardedVideoAd = uni.createRewardedVideoAd(this.adOption);
                rewardedVideoAd.onLoad(() => {
                    this.loading = false;
                    this.loadError = false;
                    console.log('onLoad event')
                });
                rewardedVideoAd.onClose((res) => {
                    this.loading = true;
                    // 用户点击了【关闭广告】按钮
                    if (res && res.isEnded) {
                        // 正常播放结束
                        console.log("onClose " + res.isEnded);
                    } else {
                        // 播放中途退出
                        console.log("onClose " + res.isEnded);
                    }

                    setTimeout(() => {
                        uni.showToast({
                            title: "激励视频" + (res.isEnded ? "成功" : "未") + "播放完毕",
                            duration: 10000,
                            position: 'bottom'
                        })
                    }, 500)
                });
                rewardedVideoAd.onError((err) => {
                    this.loading = false;
                    if (err.code && ERROR_CODE.indexOf(err.code) !== -1) {
                        this.loadError = true;
                    }
                    console.log('onError event', err)
                });
                this.loading = true;
				
            },
			viewTab(e) {
				//console.log("viewTab x"+e.touches[0].clientX+"y:"+e.touches[0].clientY)
				//console.log("viewTab x"+event.offsetX+"  y:"+event.offsetY)
				console.log("viewTab x"+(e.detail.x-e.target.offsetLeft) +"  y:"+(e.detail.y-e.target.offsetLeft))
				
			},
			async share(e) {
				wx.showShareMenu({
				  withShareTicket: true,
				  menus: e,
				  success: (res) => {
				  	console.log("success"+res);
				  },
				  fail:(res)=>{
					  console.log(res);
				  }
				})
			},
            show() {
                const rewardedVideoAd = this.rewardedVideoAd;
                rewardedVideoAd.show().catch(() => {
                    rewardedVideoAd.load()
                        .then(() => rewardedVideoAd.show())
                        .catch(err => {
                            console.log('激励视频 广告显示失败', err)
                            uni.showToast({
                                title: err.errMsg || err.message,
                                duration: 5000,
                                position: 'bottom'
                            })
                        })
                })
            },
            reloadAd() {
                this.loading = true;
                this.rewardedVideoAd.load();
            },
			setListData() {
				let page= this.max+1;
				let requrl="https://www.xuechegoo.com/api/video/recommend?page="+page;
				console.log("requre url"+requrl);
				uni.request({
					url:requrl,
					success:(res)=>{
						if(res.statusCode!=200){
							this.loadMoreText = "服务器维护中!"
							this.showLoadMore = true;
							return;
						}
						//console.log(res);
						if(res.data.length>0)
							this.data = this.data.concat(res.data);
						if(res.data.length<10)
							this.loadMoreText = "没有更多数据了!"
						this.max = this.max+1;
					},
					fail:(res)=>{
						this.loadMoreText = res
					}
				});
			},
			initData(max){
				setTimeout(() => {
					if(!max) max=0;
					this.max = max;
					this.data = [];
					this.setListData();
					uni.stopPullDownRefresh();
				}, 300);
			},
			imgclick(it){
				let newurl="/pages/API/rewarded-video-ad/rewarded-video-ad"
				//this.activeOpen=it;
				this.setActiveOpen(it);
				console.log("newurl"+newurl);
				uni.redirectTo({
						url:newurl,
						fail(e) {
							console.log(e)
						}
				})
			}
        }
    }
</script>

<style>
	.uni-common-mt{
		text-align: center;
	}
    .btn {
        margin-bottom: 20px;
    }

    .ad-tips {
        color: #999;
        padding: 30px 0;
        text-align: center;
    }
	.video {
	    flex: 1;
	    width: 100%;
		height: 500rpx;
	}
	.uni-media-list{
		padding: 5px 10px;
	}
	.uni-media-list-logo{
		height: 120rpx;
		width: 165rpx;
		margin-right: 20rpx;
	}
	.uni-media-list-body {
		height: auto;
		justify-content: space-around;
	}
	.uni-media-list-text-bottom{
		text-align: left;
	}
</style>
