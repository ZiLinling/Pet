<template>
	<view>
		<view class="status" :style="{ opacity: afterHeaderOpacity }"></view>
		<view class="header">
			<!-- 头部-默认显示 -->
			<view class="before" :style="{ opacity: 1 - afterHeaderOpacity, zIndex: beforeHeaderzIndex }">
				<view class="back">
					<view class="icon xiangqian" @tap="back" v-if="showBack"></view>
				</view>
				<view class="middle"></view>
				<view class="icon-btn">
					<view class="icon tongzhi" @tap="toMsg"></view>
					<view class="icon cart" @tap="join"></view>
				</view>
			</view>
			<!-- 头部-滚动渐变显示 -->
			<view class="after" :style="{ opacity: afterHeaderOpacity, zIndex: afterHeaderzIndex }">
				<view class="back">
					<view class="icon xiangqian" @tap="back" v-if="showBack"></view>
				</view>
				<view class="middle">
					<view v-for="(anchor,index) in anchorlist" :class="[selectAnchor==index ?'on':'']" :key="index"
						@tap="toAnchor(index)">{{anchor.name}}</view>
				</view>
				<view class="icon-btn">
					<view class="icon tongzhi" @tap="toMsg"></view>
					<view class="icon cart" @tap="join"></view>
				</view>
			</view>
		</view>
		<!-- 底部菜单 -->
		<view class="footer">
			<view class="icons">
				<view class="box" @tap="share">
					<view class="icon fenxiang"></view>
					<view class="text">分享</view>
				</view>
				<view class="box" @tap="toChat">
					<view class="icon kefu"></view>
					<view class="text">客服</view>
				</view>
				<view class="box" @tap="keep">
					<view class="icon" :class="[isKeep?'shoucangsel':'shoucang']"></view>
					<view class="text">{{isKeep?'已':''}}收藏</view>
				</view>
			</view>
			<view class="btn">
				<view class="joinCart" @tap="join()">加入购物车</view>
				<view class="buy" @tap="buy">立即购买</view>
			</view>
		</view>
		<!-- share弹窗 -->
		<view class="share" :class="shareClass" @touchmove.stop.prevent="discard" @tap="hideShare">
			<view class="mask"></view>
			<view class="layer" @tap.stop="discard">
				<view class="h1">分享</view>
				<view class="list">
					<view class="box">
						<image src="/static/img/share/wx.png"></image>
						<view class="title">
							微信好友
						</view>
					</view>
					<view class="box">
						<image src="/static/img/share/pyq.png"></image>
						<view class="title">
							朋友圈
						</view>
					</view>
					<view class="box">
						<image src="/static/img/share/wb.png"></image>
						<view class="title">
							新浪微博
						</view>
					</view>
					<view class="box">
						<image src="/static/img/share/qq.png"></image>
						<view class="title">
							QQ
						</view>
					</view>
				</view>
				<view class="btn" @tap="hideShare">
					取消
				</view>
			</view>

		</view>
		<!-- 服务-模态层弹窗 -->
		<view class="popup service" :class="serviceClass" @touchmove.stop.prevent="discard" @tap="hideService">
			<!-- 遮罩层 -->
			<view class="mask"></view>
			<view class="layer" @tap.stop="discard">
				<view class="content">
					<view class="row" v-for="(item,index) in service" :key="index">
						<view class="title">{{item.name}}</view>
						<view class="description">{{item.description}}</view>
					</view>
				</view>
				<view class="btn">
					<view class="button" @tap="hideService">完成</view>
				</view>
			</view>
		</view>
		<!-- 规格-模态层弹窗 -->
		<view class="popup spec" :class="specClass" @touchmove.stop.prevent="discard" @tap="hideSpec(0)">
			<!-- 遮罩层 -->
			<view class="mask"></view>
			<view class="layer" @tap.stop="discard">
				<view class="content">
					<view class="title">
						<view style="float: right; font-size: 24px; color: red; font-weight: 800;">
							价格: <view class="" style="float: right; font-size: 26px;">
								￥{{goodsData.price}}
							</view>
						</view>
						<view style="margin-top:30upx; margin-left: 20upx;">
							<img :src="$base_url+goodsData.img" alt="" style="border-radius: 50upx; width: 200upx;">
						</view>
					</view>
					<view class="length">
						<view class="text">数量</view>
						<view class="num">
							<view class="sub" @tap.stop="sub">
								<view class="icon jian"></view>
							</view>
							<view class="input" @tap.stop="discard">
								<input type="num" v-model="num" />
							</view>
							<view class="add" @tap.stop="add">
								<view class="icon jia"></view>
							</view>
						</view>
					</view>
				</view>
				<view class="btn">
					<view class="button" @tap="hideSpec()">完成</view>
				</view>
			</view>
		</view>
		<!-- 商品主图轮播 -->
		<view class="swiper-box">
			<swiper circular="true" autoplay="true" @change="swiperChange">
				<swiper-item v-for="swiper in swiperList" :key="swiper.id">
					<image :src="$base_url+swiper.img"></image>
				</swiper-item>
			</swiper>
			<view class="indicator">{{currentSwiper+1}}/{{swiperList.length}}</view>
		</view>
		<!-- 标题 价格 -->
		<view class="info-box goods-info">
			<view class="price">￥{{goodsData.price}}</view>
			<view class="title">
				{{goodsData.name}}
			</view>
		</view>
		<!-- 服务-规则选择 -->
		<view class="info-box spec">
			<view class="row" @tap="showService">
				<view class="text">服务</view>
				<view class="content">
					<view class="serviceitem" v-for="(item,index) in service" :key="index">{{item.name}}
					</view>
				</view>
				<view class="arrow">
					<view class="icon xiangyou"></view>
				</view>
			</view>

			<view class="row">
				<view class="text">运费：8-24 <view class="arrow"> 库存：{{goodsData.stock}}</view>
				</view>

			</view>

		</view>
		<!-- 评价 -->
		<view class="info-box comments" id="comments">
			<view class="row">
				<view class="text">商品评价({{comment.num}})</view>
				<view class="arrow" @tap="toRatings">
					<view class="show" @tap="showComments(goodsData.id)">
						查看全部
						<view class="icon xiangyou"></view>
					</view>
				</view>
			</view>
			<view class="comment" @tap="toRatings">
				<view class="user-info">
					<view class="face">
						<image :src="$base_url+comment.etc.user.img"></image>
					</view>
					<view class="username">{{comment.etc.user.name}}</view>
				</view>
				<view class="content">
					{{comment.comment}}
				</view>
			</view>
		</view>

		<view class="info-box comments" id="comments">
			<view class="row">
				<view class="text">{{store.name}}</view>
				<view class="arrow">
					<view class="show" @tap="toStore">
						查看商店
						<view class="icon xiangyou"></view>
					</view>
				</view>
			</view>

			<image :src="$base_url+store.img" @tap="toStore" mode="aspectFill" style="width: 100%;"></image>
		</view>
		<!-- 详情 -->
		<view class="description">
			<view class="title">———— 商品详情 ————</view>
			<view class="goods-description">
				<!-- 商品详情在这，看后面怎么用？ -->
				{{goodsData.description}}
			</view>
			<view class="content"><rich-text :nodes="descriptionStr"></rich-text></view>
		</view>
	</view>
</template>

<script>
	import {
		saveMsg
	}
	from '@/api/msg';
	import {
		save
	} from '@/api/cart';
	import {
		getGoods
	} from '@/api/goods';
	import {
		checkFavor,
		addFavor,
		deleteFavor
	} from '@/api/favor';
	import {
		getById
	}
	from '@/api/store';
	import {
		getListByGoodsId,
		getBestComment
	} from '@/api/comment.js'
	export default {
		data() {
			return {
				id: '',
				favorId: '',
				num: null,
				token: '',
				//控制渐变标题栏的参数
				beforeHeaderzIndex: 11, //层级
				afterHeaderzIndex: 10, //层级
				beforeHeaderOpacity: 1, //不透明度
				afterHeaderOpacity: 0, //不透明度
				//是否显示返回按钮
				// #ifndef MP
				showBack: true,
				// #endif
				//轮播主图数据
				swiperList: [{
						id: 1,
						img: '/resource/pet/cat2.jpg'
					},
					{
						id: 2,
						img: '/resource/pet/cat3.jpg'
					},
					{
						id: 3,
						img: '/resource/pet/cat6.jpg'
					},
					{
						id: 4,
						img: '/resource/pet/cat5.jpg'
					}
				],
				//轮播图下标
				choose: null,
				currentSwiper: 0,
				anchorlist: [], //导航条锚点
				selectAnchor: 0, //选中锚点
				serviceClass: '', //服务弹窗css类，控制开关动画
				specClass: '', //规格弹窗css类，控制开关动画
				shareClass: '', //分享弹窗css类，控制开关动画
				// 商品信息
				goodsData: {
					id: 1,
					name: "",
					price: "127.00",
					num: 1,
					stock: 1,
					category: '',
					description: '',
					storeName: ''
				},
				store: [],
				comment: {
					num: 102,
					userface: '/static/img/face.jpg',
					username: '大黑哥',
					content: '很不错，之前买了很多次了，很好看，能放很久，和图片色差不大，值得购买！'
				},
				service: [{
						name: "正品保证",
						description: "此商品官方保证为正品"
					},
					{
						name: "极速退款",
						description: "此商品享受退货极速退款服务"
					},
					{
						name: "7天退换",
						description: "此商品享受7天无理由退换服务"
					}
				],
				isKeep: false, //收藏
				//商品描述html
				descriptionStr: '<div style="text-align:center;"><img width="100%" src="https://ae01.alicdn.com/kf/HTB1t0fUl_Zmx1VjSZFGq6yx2XXa5.jpg"/><img width="100%" src="https://ae01.alicdn.com/kf/HTB1LzkjThTpK1RjSZFKq6y2wXXaT.jpg"/><img width="100%" src="https://ae01.alicdn.com/kf/HTB18dkiTbvpK1RjSZPiq6zmwXXa8.jpg"/></div>'
			};
		},
		onLoad(option) {
			// #ifdef MP
			//小程序隐藏返回按钮
			this.showBack = false;
			// #endif
			//option为object类型，会序列化上个页面传递的参数
			this.getgoods(option.cid);
			this.id = option.cid
			this.checkFavor()
			this.getgoods(option.cid);
			getBestComment({goodsId:option.cid}).then((response)=>{
		
				this.comment=response.data.data
						console.log(this.comment)
			}).catch((err)=>{
				console.log(err)
			})
		},
		onShow() {
			this.token = uni.getStorageSync('token')
			
		},
		onReady() {
			this.calcAnchor(); //计算锚点高度，页面数据是ajax加载时，请把此行放在数据渲染完成事件中执行以保证高度计算正确
		},
		onPageScroll(e) {
			//锚点切换
			this.selectAnchor = e.scrollTop >= this.anchorlist[2].top ? 2 : e.scrollTop >= this.anchorlist[1].top ? 1 : 0;
			//导航栏渐变
			let tmpY = 375;
			e.scrollTop = e.scrollTop > tmpY ? 375 : e.scrollTop;
			this.afterHeaderOpacity = e.scrollTop * (1 / tmpY);
			this.beforeHeaderOpacity = 1 - this.afterHeaderOpacity;
			//切换层级
			this.beforeHeaderzIndex = e.scrollTop > 0 ? 10 : 11;
			this.afterHeaderzIndex = e.scrollTop > 0 ? 11 : 10;
		},
		//上拉加载，需要自己在page.json文件中配置"onReachBottomDistance"
		onReachBottom() {
			//滑动函数？
		},
		mounted() {
			
		},
		// watch:{
		// 	goods.num	
		// },
		methods: {

			checkFavor() {
				checkFavor(this.id, 2).then((response) => {
					this.favorId = response.data.data.id
					this.isKeep = true
				}).catch((error) => {
					this.isKeep = false

				})
			},

			toStore() {
				uni.navigateTo({
					url: '/pages/store/store?cid=' + this.goodsData.storeId
				})

			},
			getgoods(goodsId) {
				getGoods({
					id: goodsId
				}).then((response) => {
					this.goodsData = response.data.data;
					this.num = 1;
					getById(this.goodsData.storeId).then((response) => {
						this.goodsData.storeName = response.data.data.name;
						this.store = response.data.data
					})
				})
			},
			//轮播图指示器
			swiperChange(event) {
				this.currentSwiper = event.detail.current;
			},
			//消息列表
			toMsg() {
				uni.navigateTo({
					url: '../msg/msg'
				})
			},
			// 客服
			toChat() {
				//add
				saveMsg({
					type: 0,
					recipient: this.store.id
				}).then((response) => {
					console.log(response)
				}).catch((error) => {
					console.log(error)
				})
				uni.navigateTo({
					url: "/pages/msg/chat/chat?recipient" + this.store.id
				})
			},
			// 分享
			share() {
				this.shareClass = 'show';
			},
			hideShare() {
				this.shareClass = 'hide';
				setTimeout(() => {
					this.shareClass = 'none';
				}, 150);
			},
			//收藏
			keep() {
				if (this.isKeep == true) {
					deleteFavor(this.favorId).then((response) => {
						this.isKeep = false
						this.checkFavor()
					})
				} else if (this.isKeep == false) {
					addFavor(this.id, 2).then((response) => {
						this.isKeep = true
						this.checkFavor()
					})
				}
			},
			// 加入购物车
			joinCart() {
				save({
					goodsId: this.goodsData.id,
					num: this.num
				}).then((response) => {}).catch((error) => {})
			},
			join() {
				this.choose = 0
				this.showSpec()
			},
			//立即购买
			buy() {
				this.choose = 1
				this.showSpec()
			},
			//商品评论
			toRatings() {
				uni.navigateTo({
					url: 'ratings/ratings'
				})
			},
			//跳转确认订单页面
			toConfirmation() {

				this.goodsData.num = this.num
				uni.setStorage({
					key: 'goodsOrder',
					data: this.goodsData,
					success: function() {
						uni.navigateTo({
							url: '/pages/order/confirmation?type=1'
						})
					}
				})
			},
			//跳转评论列表
			showComments(goodsid) {
				uni.navigateTo({
					url:'/pages/item/ratings/ratings?goodsId='+goodsid
				})
			},
			//减少数量
			sub() {

				if (this.num <= 1) {
					uni.showToast({
						title:'不能小于0'
					})
					return;
				}
				this.num--;
			},
			//增加数量
			add() {
				if(this.num>=this.goodsData.stock){
					//超出库存
					uni.showToast({
						title:'超出库存'
					})
					
					return;
				}
				this.num++;
			},
			//跳转锚点
			toAnchor(index) {
				this.selectAnchor = index;
				uni.pageScrollTo({
					scrollTop: this.anchorlist[index].top,
					duration: 200
				});
			},
			//计算锚点高度
			calcAnchor() {
				this.anchorlist = [{
						name: '主图',
						top: 0
					},
					{
						name: '评价',
						top: 0
					},
					{
						name: '详情',
						top: 0
					}
				]
				let commentsView = uni.createSelectorQuery().select("#comments");
				commentsView.boundingClientRect((data) => {
					let statusbarHeight = 0;
					//APP内还要计算状态栏高度
					// #ifdef APP-PLUS
					statusbarHeight = plus.navigator.getStatusbarHeight()
					// #endif
					let headerHeight = uni.upx2px(100);
					this.anchorlist[1].top = data.top - headerHeight - statusbarHeight;
					this.anchorlist[2].top = data.bottom - headerHeight - statusbarHeight;

				}).exec();
			},
			//返回上一页
			back() {
				uni.navigateBack();
			},
			//服务弹窗
			showService() {

				this.serviceClass = 'show';
			},
			//关闭服务弹窗
			hideService() {
				this.serviceClass = 'hide';

				setTimeout(() => {
					this.serviceClass = 'none';
				}, 200);
			},
			//规格弹窗
			showSpec(fun) {
				this.specClass = 'show';
			},
			//关闭规格弹窗
			hideSpec(action) {

				//回调
				if (action === 0) {
					this.specClass = null;
					return; // 直接返回
				}
				this.specClass = 'hide';
				if (this.token == '') {
					uni.showToast({
						title: '请先登录',
						icon: 'none'
					})
					// this.specClass = null;
					setTimeout(function() {
						uni.navigateTo({
							url: '/pages/user/login/login'
						});
						// 延时1秒后执行的函数
					}, 1000);

					return;
				}
				if (this.choose == 0) {
					this.joinCart();
					uni.showToast({
						title: "已加入购物车"
					});
				} else {
					this.toConfirmation();
				}
				setTimeout(() => {
					this.specClass = 'none';
				}, 200);
			},
			discard() {
				//丢弃
			}
		}
	};
</script>

<style lang="scss">
	page {
		background-color: #f8f8f8;
	}

	@keyframes showPopup {
		0% {
			opacity: 0;
		}

		100% {
			opacity: 1;
		}
	}

	@keyframes hidePopup {
		0% {
			opacity: 1;
		}

		100% {
			opacity: 0;
		}
	}

	@keyframes showLayer {
		0% {
			transform: translateY(0);
		}

		100% {
			transform: translateY(-100%);
		}
	}

	@keyframes hideLayer {
		0% {
			transform: translateY(-100%);
		}

		100% {
			transform: translateY(0);
		}
	}

	.icon {
		font-size: 26upx;
	}

	.status {
		width: 100%;
		height: 0;
		position: fixed;
		z-index: 10;
		top: 0;
		/*  #ifdef  APP-PLUS  */
		height: var(--status-bar-height); //覆盖样式
		/*  #endif  */
		background-color: #f1f1f1;
		transition: opacity 0.05s linear;
	}

	.header {
		width: 100%;

		height: 100upx;
		display: flex;
		align-items: center;
		position: fixed;
		top: 0;
		z-index: 10;
		/*  #ifdef  APP-PLUS  */
		top: var(--status-bar-height);

		/*  #endif  */
		.before,
		.after {
			width: 92%;
			padding: 0 4%;
			height: 100upx;
			display: flex;
			align-items: center;
			position: fixed;
			top: 0;
			/*  #ifdef  APP-PLUS  */
			top: var(--status-bar-height);
			/*  #endif  */
			transition: opacity 0.05s linear;

			.back {
				width: 125upx;
				height: 60upx;
				flex-shrink: 0;

				.icon {
					margin-left: -10%;
					width: 60upx;
					height: 60upx;
					display: flex;
					align-items: center;
					justify-content: center;
					font-size: 42upx;
				}
			}

			.middle {
				width: 100%;
			}

			.icon-btn {
				width: 125upx;
				height: 60upx;
				flex-shrink: 0;
				display: flex;

				.icon {
					&:first-child {
						margin-right: 5upx;
					}

					width: 60upx;
					height: 60upx;
					display: flex;
					justify-content: center;
					align-items: center;
					font-size: 42upx;
				}
			}
		}

		.before {
			.back {
				.icon {
					color: #fff;
					background-color: rgba(0, 0, 0, 0.2);
					border-radius: 100%;
				}
			}

			.icon-btn {
				.icon {
					color: #fff;
					background-color: rgba(0, 0, 0, 0.2);
					border-radius: 100%;

				}
			}
		}

		.after {
			background-color: #f1f1f1;

			.back {
				.icon {
					color: #666;
				}
			}

			.icon-btn {
				.icon {
					color: #666;
				}
			}

			.middle {
				font-size: 32upx;
				height: 90upx;
				display: flex;
				justify-content: center;
				align-items: center;
				padding: 0 7%;

				view {
					width: (100%/3);
					padding: 0 3%;
					margin: 0 3%;
					display: flex;
					justify-content: center;
					align-items: center;

					&.on {
						margin-bottom: -4upx;
						color: #f47952;
						border-bottom: solid 4upx #f47952;
					}
				}
			}
		}
	}

	.swiper-box {
		position: relative;
		width: 100%;
		height: 100vw;

		swiper {
			width: 100%;
			height: 100vw;

			swiper-item {
				image {
					width: 100%;
					height: 100vw;
				}
			}
		}

		.indicator {
			display: flex;
			justify-content: center;
			align-items: center;
			padding: 0 25upx;
			height: 40upx;
			border-radius: 40upx;
			font-size: 22upx;
			position: absolute;
			bottom: 20upx;
			right: 20upx;
			color: #fff;
			background-color: rgba(0, 0, 0, 0.2);
		}
	}

	.info-box {
		width: 92%;
		padding: 20upx 4%;
		background-color: #fff;
		margin-bottom: 20upx;
	}

	.goods-info {
		.price {
			font-size: 46upx;
			font-weight: 600;
			color: #f47925;
		}

		.title {
			text-align: right;
			font-size: 30upx;
			color: #807c87;
			width: 100%;
			display: -webkit-box;
			/** 对象作为伸缩盒子模型显示 **/
			overflow: hidden;
			word-break: break-all;
			/* break-all(允许在单词内换行。) https://www.w3school.com.cn/cssref/pr_word-break.asp*/
			text-overflow: ellipsis;
			/* 超出部分省略号 */
			-webkit-box-orient: vertical;
			/** 设置或检索伸缩盒对象的子元素的排列方式 **/
			-webkit-line-clamp: 1;
			/** 显示的行数 **/
		}
	}

	.spec {
		.row {
			width: 100%;
			display: flex;
			align-items: center;
			margin: 0 0 30upx 0;

			.text {
				font-size: 24upx;
				color: #a2a2a2;
				margin-right: 20upx;
			}

			.content {
				font-size: 28upx;
				display: flex;
				flex-wrap: wrap;

				.serviceitem {
					margin-right: 10upx;
				}

				.sp {
					width: 100%;
					display: flex;

					view {
						background-color: #f6f6f6;
						padding: 5upx 10upx;
						color: #999;
						margin-right: 10upx;
						font-size: 20upx;
						border-radius: 5upx;

						&.on {
							border: solid 1upx #f47952;
							padding: 4upx 8upx;
						}
					}
				}
			}

			.arrow {
				position: absolute;
				right: 4%;

				.icon {
					color: #ccc;
				}
			}
		}
	}

	.comments {
		.row {
			width: 100%;
			height: 40upx;
			display: flex;
			align-items: center;
			margin: 0 0 30upx 0;

			.text {
				font-size: 30upx;
			}

			.arrow {
				font-size: 28upx;
				position: absolute;
				right: 4%;
				color: #17e6a1;

				.show {
					display: flex;
					align-items: center;

					.icon {
						color: #17e6a1;
					}
				}
			}
		}

		.comment {
			width: 100%;

			.user-info {
				width: 100%;
				height: 40upx;
				display: flex;
				align-items: center;

				.face {
					width: 40upx;
					height: 40upx;
					margin-right: 8upx;

					image {
						width: 40upx;
						height: 40upx;
						border-radius: 100%;
					}
				}

				.username {
					font-size: 24upx;
					color: #999;
				}
			}

			.content {
				margin-top: 10upx;
				font-size: 26upx;
			}
		}
	}


	.description {
		.title {
			width: 100%;
			height: 80upx;
			display: flex;
			justify-content: center;
			align-items: center;
			font-size: 26upx;
			color: #999;
		}

		.goods-description {
			display: flex;
			flex-direction: row;
			width: 100%;
			align-items: center; //垂直居中
			text-align: center;
			background-color: (248, 248, 248);
		}

		margin-bottom: 1000upx;
	}

	.footer {
		position: fixed;
		bottom: 0upx;
		width: 92%;
		padding: 0 4%;
		height: 99upx;
		border-top: solid 1upx #eee;
		background-color: #fff;
		z-index: 2;
		display: flex;
		justify-content: space-between;
		align-items: center;

		.icons {
			display: flex;
			height: 80upx;
			margin-left: -4%;

			.box {
				width: 80upx;
				height: 80upx;
				display: flex;
				justify-content: center;
				flex-wrap: wrap;

				.icon {
					font-size: 40upx;
					color: #828282;
				}

				.text {
					display: flex;
					justify-content: center;
					width: 100%;
					font-size: 22upx;
					color: #666;
				}
			}
		}

		.btn {
			height: 80upx;
			border-radius: 40upx;
			overflow: hidden;
			display: flex;
			margin-right: -2%;

			.joinCart,
			.buy {
				height: 80upx;
				padding: 0 40upx;
				color: #fff;
				display: flex;
				align-items: center;
				font-size: 28upx;
			}

			.joinCart {
				background-color: #f0b46c;
			}

			.buy {
				background-color: #f06c7a;
			}
		}
	}

	.popup {
		position: fixed;
		top: 0;
		width: 100%;
		height: 100%;
		z-index: 20;
		display: none;

		.mask {
			position: fixed;
			top: 0;
			width: 100%;
			height: 100%;
			z-index: 21;
			background-color: rgba(0, 0, 0, 0.6);
		}

		.layer {
			position: fixed;
			z-index: 22;
			bottom: -70%;
			width: 92%;
			padding: 0 4%;
			height: 70%;
			border-radius: 20upx 20upx 0 0;
			background-color: #fff;
			display: flex;
			flex-wrap: wrap;
			align-content: space-between;

			.content {
				width: 100%;
				padding: 20upx 0;
			}

			.btn {
				width: 100%;
				height: 100upx;

				.button {
					width: 100%;
					height: 80upx;
					border-radius: 40upx;
					color: #fff;
					display: flex;
					align-items: center;
					justify-content: center;
					background-color: #f47952;
					font-size: 28upx;
				}
			}
		}

		&.show {
			display: block;

			.mask {
				animation: showPopup 0.2s linear both;
			}

			.layer {
				animation: showLayer 0.2s linear both;
			}
		}

		&.hide {
			display: block;

			.mask {
				animation: hidePopup 0.2s linear both;
			}

			.layer {
				animation: hideLayer 0.2s linear both;
			}
		}

		&.none {
			display: none;
		}

		&.service {
			.row {
				margin: 30upx 0;

				.title {
					font-size: 30upx;
					margin: 10upx 0;
				}

				.description {
					font-size: 28upx;
					color: #999;
				}
			}
		}

		&.spec {
			.title {
				font-size: 30upx;
				margin: 30upx 0;
			}

			.sp {
				display: flex;

				view {
					font-size: 28upx;
					padding: 5upx 20upx;
					border-radius: 8upx;
					margin: 0 30upx 20upx 0;
					background-color: #f6f6f6;

					&.on {
						padding: 3upx 18upx;
						border: solid 1upx #f47925;
					}
				}
			}

			.length {
				margin-top: 30upx;
				border-top: solid 1upx #aaa;
				display: flex;
				justify-content: space-between;
				align-items: center;
				padding-top: 20upx;

				.text {
					font-size: 30upx;
				}

				.num {
					display: flex;
					justify-content: center;
					align-items: center;

					.input {
						width: 80upx;
						height: 60upx;
						margin: 0 10upx;
						background-color: #f3f3f3;
						display: flex;
						justify-content: center;
						align-items: center;
						text-align: center;

						input {
							width: 80upx;
							height: 60upx;
							display: flex;
							justify-content: center;
							align-items: center;
							text-align: center;
							font-size: 26upx;
						}
					}

					.sub,
					.add {
						width: 60upx;
						height: 60upx;
						background-color: #f3f3f3;
						border-radius: 5upx;

						.icon {
							font-size: 30upx;
							width: 60upx;
							height: 60upx;
							display: flex;
							justify-content: center;
							align-items: center;

						}
					}
				}
			}

		}
	}

	.share {
		display: none;

		&.show {
			display: block;

			.mask {
				animation: showPopup 0.15s linear both;
			}

			.layer {
				animation: showLayer 0.15s linear both;
			}
		}

		&.hide {
			display: block;

			.mask {
				animation: hidePopup 0.15s linear both;
			}

			.layer {
				animation: hideLayer 0.15s linear both;
			}
		}

		&.none {
			display: none;
		}

		.mask {
			background-color: rgba(0, 0, 0, .5);
			position: fixed;
			width: 100%;
			height: 100%;
			top: 0;
			z-index: 11;
		}

		.layer {
			width: 92%;
			position: fixed;
			z-index: 12;
			padding: 0 4%;
			top: 100%;
			background-color: rgba(255, 255, 255, 0.9);

			.list {
				width: 100%;
				display: flex;
				padding: 10upx 0 30upx 0;

				.box {
					width: 25%;
					display: flex;
					justify-content: center;
					flex-wrap: wrap;

					image {
						width: 13.8vw;
						height: 13.8vw;
					}

					.title {
						margin-top: 10upx;
						display: flex;
						justify-content: center;
						width: 100%;
						font-size: 26upx;
					}
				}
			}

			.btn {
				width: 100%;
				height: 100upx;
				display: flex;
				justify-content: center;
				align-items: center;
				font-size: 28upx;
				border-top: solid 1upx #666666;
			}

			.h1 {
				width: 100%;
				height: 80upx;
				display: flex;
				justify-content: center;
				align-items: center;
				font-size: 34upx;
			}
		}
	}
</style>