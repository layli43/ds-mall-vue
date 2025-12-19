<template>
	<view class="content">
		<image src="/static/new_product_banner.png" class="banner-image"></image>
		<view class="section-tit">Related Products</view>
		
		<view class="goods-list">
			<view v-for="(item, index) in productList" :key="index" class="goods-item" @click="navToDetailPage(item)">
				<view class="image-wrapper">
					<image :src="item.pic" mode="aspectFit"></image>
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

<script>
	import {
		fetchNewProductList
	} from '@/api/home.js';
	import uniLoadMore from '@/components/uni-load-more/uni-load-more.vue';
	export default {
		components: {
			uniLoadMore
		},
		data() {
			return {
				loadingType: 'more', // "Load More" status
				productList: [],
				searchParam: {
					pageNum: 1,
					pageSize: 6
				}
			};
		},

		onLoad(options) {
			this.loadData();
		},
		// Pull-down refresh
		onPullDownRefresh() {
			this.loadData('refresh');
		},
		// Reach bottom to load more
		onReachBottom() {
			this.searchParam.pageNum++;
			this.loadData();
		},
		methods: {
			/**
			 * Load products with pull-to-refresh and infinite scroll
			 * @param {String} type 'add' for loading more, 'refresh' for resetting
			 * @param {Number} loading Loading indicator toggle
			 */
			async loadData(type = 'add', loading) {
				// If no more data, return immediately
				if (type === 'add') {
					if (this.loadingType === 'nomore') {
						return;
					}
					this.loadingType = 'loading';
				} else {
					this.loadingType = 'more'
				}

				if (type === 'refresh') {
					this.searchParam.pageNum = 1;
					this.productList = [];
				}
				
				fetchNewProductList(this.searchParam).then(response => {
					let productList = response.data;
					
					if (response.data.length === 0) {
						// No more products found
						this.loadingType = 'nomore';
						this.searchParam.pageNum--;
					} else {
						if (response.data.length < this.searchParam.pageSize) {
							// Result set is smaller than page size, no more pages
							this.loadingType = 'nomore';
							this.searchParam.pageNum--;
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
			// Navigate to Product Detail
			navToDetailPage(item) {
				let id = item.id;
				uni.navigateTo({
					url: `/pages/product/product?id=${id}`
				})
			},
			stopPrevent() {}
		},
	}
</script>

<style lang="scss">
	/* Product List Styles */
    /* (Styles remain the same as they use CSS properties) */
	page,
	.content {
		background: $page-color-base;
	}
	.banner-image{
		width: 100%;
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
			background-color: #fff;

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