<template>
		<view class="uni-padding-wrap uni-common-mt">
			<view class="text" v-for="item,index in data" :key="item.id">
				<rich-text :nodes="item.title" style="word-break:break-all;"></rich-text>
			    <image class="img" :src="item.imgsrc" @click="imgclick(item )"></image>
				<view style="display: flex;">
					<text style="font-size: small;margin-top: 10rpx;">播放次数：{{item.playcnt}}</text>
					<button :data-name="index" style="width: 300rpx; margin-right: 20rpx;"  open-type="share">分享给好友</button>
				</view>
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
	export default {
		data() {
			return {
				data: [],
				loadMoreText: "加载中...",
				showLoadMore: false,
				max: 0
			}
		},
		onLoad() {
			this.initData(); 
		},
		onUnload() {
			this.max = 0,
			this.data = [],
			this.loadMoreText = "加载更多",
			this.showLoadMore = false;
		},
		onReachBottom() {
			console.log("onReachBottom");
			this.showLoadMore = true;
			setTimeout(() => {
				this.setListData();
			}, 300);
		},
		onPullDownRefresh() {
			console.log('onPullDownRefresh');
			let tt=Math.floor((Math.random()*10)+1);
			this.initData(tt);
		},
		onShareAppMessage(options) {
			let inde=  0;
			if( options.from == 'button' ){
			　　　　var eData = options.target.dataset;
				   inde=eData.name
			　　　　console.log("onShareAppMessage"+inde) 
			}
			let it=  this.data[inde];
			this.setActiveOpen(it);
			let newurl="/pages/API/rewarded-video-ad/rewarded-video-ad?src="+it.src;
			console.log("share newurl"+newurl);
			return {
			  title:it.title,
			  path: newurl,
			  imageUrl:it.imgsrc
			}
		},
		methods: {
			...mapMutations(['setActiveOpen']),
			imgclick(it){
				this.setActiveOpen(it);
				// let newurl="/pages/API/rewarded-video-ad/rewarded-video-ad?src="+it.src;
				// newurl= newurl+"&id="+it.id;
				let newurl="/pages/API/rewarded-video-ad/rewarded-video-ad";
				uni.navigateTo({
						url:newurl,
						fail(e) {
							console.log(e)
						}
				})
			},
			setListData() {
				let page= this.max+1;
				let requrl="https://www.xuechegoo.com/api/video/list?page="+page;
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
			}
		}
	}
</script>

<style>
	.uni-padding-wrap{
		background-color: #c8c0c4;
		margin-top: 0;
		padding: 10rpx  5rpx;
	},
	.text {
		margin: 20rpx 0;
		padding: 10rpx;
		width:97%;
		background-color: #fff;
		border-radius: 10rpx;
	},
	.img{
		width: 97%;
		margin: 5rpx 0;
	}
</style>
