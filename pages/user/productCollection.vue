<template>
	<view class="content">
		<!-- Empty page -->
		<empty v-if="productList == null || productList.length === 0"></empty>

		<view class="hot-section">
			<view
				v-for="(item, index) in productList"
				:key="index"
				class="guess-item"
				@click="navToDetailPage(item)">
				
				<view class="image-wrapper">
					<image :src="item.productPic" mode="aspectFill"></image>
				</view>

				<view class="txt">
					<text class="title clamp">{{ item.productName }}</text>
					<text class="title2">{{ item.productSubTitle }}</text>
					<view class="hor-txt">
						<text class="price">￥{{ item.productPrice }}</text>
					</view>
				</view>
			</view>
		</view>

		<uni-load-more :status="loadingType"></uni-load-more>
	</view>
</template>

<script>
	import empty from "@/components/empty";
	import uniLoadMore from '@/components/uni-load-more/uni-load-more.vue';
	import { formatDate } from '@/utils/date';
	import {
		fetchProductCollectionList,
		clearProductCollection
	} from '@/api/memberProductCollection.js';

	export default {
		components: {
			uniLoadMore,
			empty
		},
		data() {
			return {
				loadingType: 'more',
				productList: [],
				searchParam: {
					pageNum: 1,
					pageSize: 6
				}
			};
		},
		onLoad() {
			this.loadData();
		},

		// Pull-down refresh
		onPullDownRefresh() {
			this.loadData('refresh');
		},

		// Load more when reaching bottom
		onReachBottom() {
			this.searchParam.pageNum++;
			this.loadData();
		},

		// #ifndef MP
		onNavigationBarButtonTap(e) {
			const index = e.index;
			const self = this;
			if (index === 0) {
				uni.showModal({
					title: 'Notice',
					content: 'Do you want to clear all collections?',
					success(res) {
						if (res.confirm) {
							clearProductCollection().then(() => {
								self.loadData('refresh');
							});
						}
					}
				});
			}
		},
		// #endif

		filters: {
			formatDateTime(time) {
				if (!time) {
					return 'N/A';
				}
				const date = new Date(time);
				return formatDate(date, 'yyyy-MM-dd hh:mm:ss');
			}
		},

		methods: {
			// Load products with pull-down refresh and scroll-up loading
			async loadData(type = 'add', loading) {
				// No more data, return directly
				if (type === 'add') {
					if (this.loadingType === 'nomore') {
						return;
					}
					this.loadingType = 'loading';
				} else {
					this.loadingType = 'more';
				}

				if (type === 'refresh') {
					this.searchParam.pageNum = 1;
					this.productList = [];
				}

				fetchProductCollectionList(this.searchParam).then(response => {
					const dataList = response.data.list;

					if (dataList.length === 0) {
						// No more data
						this.loadingType = 'nomore';
						this.searchParam.pageNum--;
					} else {
						if (dataList.length < this.searchParam.pageSize) {
							this.loadingType = 'nomore';
							this.searchParam.pageNum--;
						} else {
							this.loadingType = 'more';
						}
						this.productList = this.productList.concat(dataList);
					}

					if (type === 'refresh') {
						if (loading === 1) {
							uni.hideLoading();
						} else {
							uni.stopPullDownRefresh();
						}
					}
				});
			},

			// Navigate to product detail page
			navToDetailPage(item) {
				const id = item.productId;
				uni.navigateTo({
					url: `/pages/product/product?id=${id}`
				});
			},

			stopPrevent() {}
		}
	};
</script>

<style lang="scss">
	page,
	.content {
		background: $page-color-base;
	}

	.hot-section {
		display: flex;
		flex-wrap: wrap;
		padding: 0 30upx;
		margin-top: 16upx;
		background: #fff;

		.guess-item {
			display: flex;
			flex-direction: row;
			width: 100%;
			padding-bottom: 40upx;
		}

		.image-wrapper {
			width: 30%;
			height: 250upx;
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

		.price {
			font-size: $font-lg;
			color: $uni-color-primary;
			line-height: 80upx;
		}

		.txt {
			width: 70%;
			display: flex;
			flex-direction: column;
			padding-left: 40upx;
		}

		.hor-txt {
			display: flex;
			justify-content: space-between;
		}
	}
</style>
