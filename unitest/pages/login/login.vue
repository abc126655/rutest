<template>
	<view>
		<view class="uni-padding-wrap">
			<button style="background-color: #FFFFFF;"><image :src="infourl" class="headimg" ></image></button>
			<view style="background:#FFF; padding:40rpx;">
				<block v-if="hasLogin === true">
					<view class="uni-h3 uni-center uni-common-mt">{{infoLogin}}
						<text v-if="isUniverifyLogin" style="font-size: 0.8em;">
							<i v-if="!phoneNumber.length" class="uni-icon_toast uni-loading"></i>
							<i v-else>（{{phoneNumber}}）</i>
						</text> 
					</view>
					<view class="uni-hello-text uni-center">
						<text>每个账号仅需登录 1 次，\n后续每次进入页面即可自动拉取用户信息。</text>
					</view>
				</block>
				<block v-if="hasLogin === false">
					<view class="uni-h3 uni-center uni-common-mt">未登录</view>
					<view class="uni-hello-text uni-center">
						请点击按钮登录
					</view>
				</block>
			</view>
			<view class="uni-btn-v uni- uni-common-mt">
				<!-- #ifdef MP-TOUTIAO -->
				<button type="primary" class="page-body-button" v-for="(value,key) in providerList" @click="tologin(value)" :key="key">
					登录
				</button>
				<!-- #endif -->
				<!-- #ifndef MP-TOUTIAO -->
				<button type="primary" class="page-body-button" v-for="(value,key) in providerList" @click="tologin(value)"
				 :loading="value.id === 'univerify' ? univerifyBtnLoading : false" :key="key">{{value.name}}</button>
				<!-- #endif -->
			</view>
		</view>
	</view>
</template>
<script>
	import {
		mapState,
		mapMutations,
		mapActions
	} from 'vuex'
	const univerifyInfoKey = 'univerifyInfo';

	export default {
		data() {
			return {
				title: 'login',
				infoLogin:"已经登陆",
				infourl:"../../static/extui.png",
				providerList: [],
				phoneNumber: '',
				univerifyBtnLoading: false
			}
		},
		computed: {
			...mapState(['hasLogin', 'isUniverifyLogin', 'univerifyErrorMsg'])
		},
		onLoad() {
			uni.getProvider({
				service: 'oauth',
				success: (result) => {
					this.providerList = result.provider.map((value) => {
						let providerName = '';
						switch (value) {
							case 'weixin':
								providerName = '微信登录'
								break;
							case 'qq':
								providerName = 'QQ登录'
								break;
							case 'sinaweibo':
								providerName = '新浪微博登录'
								break;
							case 'xiaomi':
								providerName = '小米登录'
								break;
							case 'alipay':
								providerName = '支付宝登录'
								break;
							case 'baidu':
								providerName = '百度登录'
								break;
							case 'toutiao':
								providerName = '头条登录'
								break;
							case 'apple':
								providerName = '苹果登录'
								break;
							case 'univerify':
								providerName = '一键登录'
								break;
						}
						return {
							name: providerName,
							id: value
						}
					});

				},
				fail: (error) => {
					console.log('获取登录通道失败', error);
				}
			});

			if (this.hasLogin && this.isUniverifyLogin) {
				this.getPhoneNumber(uni.getStorageSync(univerifyInfoKey)).then((phoneNumber) => {
					this.phoneNumber = phoneNumber
				})
			}
		},
		methods: {
			...mapMutations(['login', 'setUniverifyLogin']),
			...mapActions(['getPhoneNumber']),
			Toast(data, duration = 1000) {
				uni.showToast(Object.assign({}, data, {
					duration
				}))
			},
			tologin(provider) {
				if (provider.id === 'univerify') {
					this.univerifyBtnLoading = true;
				}

				// 一键登录已在APP onLaunch的时候进行了预登陆，可以显著提高登录速度。登录成功后，预登陆状态会重置
				// #ifdef MP-WEIXIN
				  // 获取用户信息
				  uni.getUserProfile({
					desc: provider.id,
					success: async(res)=> {
					  console.log('用户昵称为：' + res.rawData);
					  this.Toast({
					  	title: res.userInfo.nickName+ '登录成功'
					  })
					  this.infoLogin=res.userInfo.nickName+ '登录成功'
					  this.infourl =res.userInfo.avatarUrl;
					  // 更新保存在 store 中的登录状态
					  this.login(provider.id);
					},
					fail:async(res)=>{
					  console.log(res)
					}
				   });
				
				// #endif
			},
			loginByUniverify(provider, res) {
				this.setUniverifyLogin(true);
				uni.closeAuthView();

				const univerifyInfo = {
					provider,
					...res.authResult,
				}

				this.getPhoneNumber(univerifyInfo).then((phoneNumber) => {
					this.phoneNumber = phoneNumber;
					uni.setStorageSync(univerifyInfoKey, univerifyInfo)
				}).catch(err => {
					uni.showModal({
						showCancel: false,
						title: '手机号获取失败',
						content: `${err.errMsg}\n，错误码：${err.code}`
					})
					console.error(res);
				})
			},
			async loginByApple(provider, res) {
				// 获取用户信息
				const [getUserInfoErr, result] = await uni.getUserInfo({
					provider
				});
				if (getUserInfoErr) {
					let content = getUserInfoErr.errMsg;
					if (~content.indexOf('uni.login')) {
						content = '请在登录页面完成登录操作';
					}
					uni.showModal({
						title: '获取用户信息失败',
						content: '错误原因' + content,
						showCancel: false
					});
				}
				// uni-id 苹果登录
				uni.request({
					url: 'https://97fca9f2-41f6-449f-a35e-3f135d4c3875.bspapp.com/http/user-center',
					method: 'POST',
					data: {
						action: 'loginByApple',
						params: result.userInfo
					},
					success: (res) => {
						console.log('uniId login success', res);
						if(res.data.code !== 0){
							uni.showModal({
								showCancel: false,
								content: `苹果登录失败: ${JSON.stringify(res.data.msg)}`,
							})
						} else {
							uni.setStorageSync('openid', res.data.openid)
							uni.setStorageSync('apple_nickname', res.data.userInfo.nickname)
						}
					},
					fail: (e) => {
						uni.showModal({
							content: `苹果登录失败: ${JSON.stringify(e)}`,
							showCancel: false
						})
					}
				})
			}
		}
	}
</script>

<style>
	button {
		background-color: #007aff;
		color: #ffffff;
	}
	.headimg{
		width: 130rpx;
		height: 130rpx;
				justify-content: center;
				border-radius: 50px;	
				border: 3rpx solid #6699FF;
	}
</style>
