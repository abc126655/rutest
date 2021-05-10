<template>
		<view class="uni-padding-wrap uni-common-mt">
			<view style="font-size: 12px; color: #666;">{{title}}</view>
			<view class="text" v-for="item in data" :key="item.id">
				<rich-text :nodes="item.tittle"></rich-text>
			    <image class="img" :src="item.imgsrc" @click="imgclick(item )"></image>
			</view> 
			<view class="uni-loadmore" v-if="showLoadMore">{{loadMoreText}}</view>
		</view>
</template>
<script>
	export default {
		data() {
			return {
				title: '下拉刷新 + 加载更多',
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
			imgclick(it){
				let newurl="/pages/API/rewarded-video-ad/rewarded-video-ad?src="+it.src;
				newurl= newurl+"&id="+it.id;
				console.log("newurl"+newurl);
				uni.navigateTo({
						url:newurl,
						fail(e) {
							console.log(e)
						}
				})
			},
			setListData() {
				let data = [];
				this.max += 10;
				for (var i = this.max - 9; i < this.max + 1; i++) {
					let it={};
					it.id="vedi"+i;
					it.tittle="<h4> tittle--"+i+"</h4>";
					it.imgsrc='/static/img.png'
					it.src='https://img.cdn.aliyun.dcloud.net.cn/guide/uniapp/hellouniapp/hello-nvue-swiper-vertical-0'+i+'.mp4';
					data.push(it)
				}
				this.data = this.data.concat(data);
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
		padding: 0  5rpx;
	},
	.text {
		margin: 16rpx 0;
		padding: 10rpx 0;
		width:100%;
		background-color: #fff;
		text-align: center;
		border-radius: 8rpx;
	},
	.img{
		width: 100%;
		margin: 0px 0;
	}
</style>
