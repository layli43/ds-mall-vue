<template>
	<view>
		<!-- Top Banner Image -->
		<view class="top-image">
			<view class="image-wrapper">
				<image :src="brand.bigPic" class="loaded" mode="aspectFill"></image>
			</view>
		</view>
		<!-- Brand Information -->
		<view class="info">
			<view class="image-wrapper">
				<image :src="brand.logo" class="loaded" mode="aspectFit"></image>
			</view>
			<view class="title">
				<text :class="{Skeleton:!loaded}">{{brand.name}}</text>
				<text :class="{Skeleton:!loaded}">Brand Initial: {{brand.firstLetter}}</text>
			</view>
			<view>
				<text class="yticon icon-shoucang" :class="{active: favoriteStatus}" @click="favorite()"></text>
			</view>
		</view>
		<!-- Brand Story -->
		<view class="section-tit">Brand Story</view>
		<view class="brand-story">
			<text class="text">{{brand.brandStory}}</text>
		</view>
		<!-- Related Products -->
		<view class="section-tit">Related Products</view>
		<view class="goods-list">
			<view v-for="(item, index) in productList" :key="index" class="goods-item" @click="navToDetailPage(item)">
				<view class="image-wrapper">
					<image :src="item.pic" mode="aspectFill"></image>
				</view>
				<text class="title clamp">{{item.name}}</text>
				<text class="title2">{{item.subTitle}}</text>
				<view class="price-box">
					<text class="price">{{item.price}}</text>
					<text>Sold {{item.sale}}</text>
				</view>
			</view>
		</view>
		<uni-load-more :status="loadingType"></uni-load-more>
	</view>
</template>

<script setup>
import share from '@/components/share';
import {
	getBrandDetail,
	fetchBrandProductList
} from '@/api/brand.js';
import {
	createBrandAttention,
	deleteBrandAttention,
	brandAttentionDetail
} from '@/api/memberBrandAttention.js';
import uniLoadMore from '@/components/uni-load-more/uni-load-more.vue';
import {
	mapState
} from 'vuex';
export default {
		components: {
			share
		},
		data() {
			return {
				loaded: false,
				brand: {},
				productList:[],
				loadingType: 'more',
				favoriteStatus:false,
				queryParam: {
					brandId: null,
					pageNum: 1,
					pageSize: 4
				}
			};
		},
		onLoad(options) {
			this.loaded = true;
			let id = options.id;
			getBrandDetail(id).then(response => {
				this.brand = response.data;
				this.initBrandAttention();
			});
			this.queryParam.brandId = id;
			this.loadData('refresh');
		},
		computed: {
			...mapState(['hasLogin'])
		},
		methods: {
			imageOnLoad(key, index) {
				this.$set(this.data[key][index], 'loaded', 'loaded');
			},
			//Favorite
			favorite() {
				if (!this.checkForLogin()) {
					return;
				}
				if (this.favoriteStatus) {
					//Cancel favorite
					deleteBrandAttention({
						brandId: this.brand.id
					}).then(response => {
						uni.showToast({
							title: "Unfavorited successfully!",
							icon: 'none'
						});
						this.favoriteStatus = !this.favoriteStatus;
					});
				} else {
					//Add to favorites
					let brandAttention = {
						brandId : this.brand.id,
						brandName : this.brand.name,
						brandLogo : this.brand.logo,
						brandCity : ""
					}
					createBrandAttention(brandAttention).then(response=>{
						uni.showToast({
							title: "Favorited successfully!",
							icon: 'none'
						});
						this.favoriteStatus = !this.favoriteStatus;
					});
				}
			},
			//Detail
			navToDetailPage(item) {
				let id = item.id;
				uni.navigateTo({
					url: `/pages/product/product?id=${id}`
				})
			},
			//Load products with pull-to-refresh and scroll-to-load
			async loadData(type = 'add', loading) {
				//Return if no more data
				if (type === 'add') {
					if (this.loadingType === 'nomore') {
						return;
					}
					this.loadingType = 'loading';
				} else {
					this.loadingType = 'more'
				}
			
				if (type === 'refresh') {
					this.queryParam.pageNum=1;
					this.productList = [];
				}
				fetchBrandProductList(this.queryParam).then(response => {
					let productList = response.data.list;
					if (response.data.list.length === 0) {
						//No more data
						this.loadingType = 'nomore';
						this.queryParam.pageNum--;
					} else {
						if (response.data.list.length < this.queryParam.pageSize) {
							this.loadingType = 'nomore';
							this.queryParam.pageNum--;
						} else {
							this.loadingType = 'more';
						}
						this.productList = this.productList.concat(productList);
					}
					if (type === 'refresh') {
						if (loading == 1) {
							uni.hideLoading()
						} else {
							uni.stopPullDownRefresh();
						}
					}
				});
			},
			//Pull down to refresh
			onPullDownRefresh() {
				this.loadData('refresh');
			},
			//Load more
			onReachBottom() {
				this.queryParam.pageNum++;
				this.loadData();
			},
			//Initialize favorite status
			initBrandAttention(){
				if(this.hasLogin){
					brandAttentionDetail({brandId:this.brand.id}).then(response=>{
						this.favoriteStatus = response.data!=null;
					});
				}
			},
			//Check login status and show login dialog
			checkForLogin() {
				if (!this.hasLogin) {
					uni.showModal({
						title: 'Notice',
						content: 'You are not logged in. Do you want to log in?',
						confirmText: 'Go to Login',
						cancelText: 'Cancel',
						success: function(res) {
							if (res.confirm) {
								uni.navigateTo({
								url: '/pages/public/login'
							})
						} else if (res.cancel) {
							console.log('User clicked cancel');
							}
						}
					});
					return false;
				} else {
					return true;
				}
			},
		}
	}
</script>

<style lang="scss">
	page {
		background: $page-color-base;
	}

	.top-image {
		height: 200px;

		.image-wrapper {
			display: flex;
			justify-content: center;
			align-content: center;
			width: 100%;
			height: 100%;
			overflow: hidden;

			image {
				width: 100%;
				height: 100%;
			}
		}
	}

	.info {
		display: flex;
		align-items: center;
		padding: 30upx 50upx;
		background: #fff;
		margin-top: 16upx;
		.image-wrapper {
			width: 210upx;
			height: 70upx;
			background: #fff;
			image{
				width:100%;
				height: 100%;
			}
		}

		.title {
			flex: 1;
			display: flex;
			flex-direction: column;
			font-size: $font-lg+4upx;
			margin-left: 30upx;
			color: $font-color-dark;
			text:last-child {
				font-size: $font-sm;
				color: $font-color-light;
				margin-top: 8upx;
				&.Skeleton{
					width:220upx;
				}
			}
		}

		.yticon {
			font-size: 80upx;
			color: $font-color-base;
			margin: 0 10upx 0 30upx;
			&.active {
				color: #ff4443;
			}
		}
	}

	.brand-story {
		display: flex;
		padding: 30upx;
		background: #fff;

		.text {
			font-size: $font-sm;
			color: $font-color-light;
		}
	}

	.actions {
		padding: 10upx 28upx;
		background: #fff;

		.yticon {
			font-size: 46upx;
			color: $font-color-base;
			padding: 10upx 12upx;

			&.active {
				color: #ff4443;
			}

			&:nth-child(2) {
				font-size: 50upx;
			}
		}
	}
	
	.section-tit {
		font-size: $font-base+2upx;
		color: $font-color-dark;
		background: #fff;
		margin-top: 16upx;
		text-align: center;
		padding-top: 20upx;
		padding-bottom: 20upx;
	}

	/* 商品列表 */
	.goods-list {
		display: flex;
		flex-wrap: wrap;
		padding: 0 30upx;
		background: #fff;
	
		.goods-item {
			display: flex;
			flex-direction: column;
			width: 48%;
			padding-bottom: 40upx;
	
			&:nth-child(2n+1) {
				margin-right: 4%;
			}
		}
	
		.image-wrapper {
			width: 100%;
			height: 330upx;
			border-radius: 3px;
			overflow: hidden;
	
			image {
				width: 100%;
				height: 100%;
				opacity: 1;
			}
		}
	
		.title {
			font-size: $font-lg;
			color: $font-color-dark;
			line-height: 80upx;
		}
	
		.title2 {
			font-size: $font-sm;
			color: $font-color-light;
			line-height: 40upx;
			height: 80upx;
			overflow: hidden;
			text-overflow: ellipsis;
			display: block;
		}
	
		.price-box {
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding-right: 10upx;
			font-size: 24upx;
			color: $font-color-light;
		}
	
		.price {
			font-size: $font-lg;
			color: $uni-color-primary;
			line-height: 1;
	
			&:before {
				content: '￥';
				font-size: 26upx;
			}
		}
	}
</style>
