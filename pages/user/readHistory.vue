<template>
	<view class="content">
		<!-- Empty page -->
		<empty v-if="productList==null || productList.length === 0"></empty>

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
					<text class="title clamp">{{item.productName}}</text>
					<text class="title2">{{item.productSubTitle}}</text>
					<view class="hor-txt">
						<text class="price">￥{{item.productPrice}}</text>
						<text class="time">{{item.createTime | formatDateTime}}</text>
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
		fetchReadHistoryList,
		clearReadHistory
	} from '@/api/memberReadHistory.js';

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
		onLoad(options) {
			this.loadData();
		},
		// Pull-down refresh
		onPullDownRefresh() {
			this.loadData('refresh');
		},
		// Load more
		onReachBottom() {
			this.searchParam.pageNum++;
			this.loadData();
		},
		// #ifndef MP
		onNavigationBarButtonTap(e) {
			const index = e.index;
			let thisObj = this;
			if (index === 0) {
				uni.showModal({
					title: 'Notice',
					content: 'Do you want to clear all browsing history?',
					success: function (res) {
						if (res.confirm) {
							clearReadHistory().then(() => {
								thisObj.loadData('refresh');
							});
						}
					}
				});
			}
		},
		// #endif
		filters: {
			formatDateTime(time) {
				if (time == null || time === '') {
					return 'N/A';
				}
				let date = new Date(time);
				return formatDate(date, 'yyyy-MM-dd hh:mm:ss')
			},
		},
		methods: {
			// Load products, with pull-down refresh and scroll-up loading
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

				fetchReadHistoryList(this.searchParam).then(response => {
					let dataList = response.data.list;
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
						if (loading == 1) {
							uni.hideLoading();
						} else {
							uni.stopPullDownRefresh();
						}
					}
				});
			},
			// Product details
			navToDetailPage(item) {
				let id = item.productId;
				uni.navigateTo({
					url: `/pages/product/product?id=${id}`
				});
			},
			stopPrevent() {}
		},
	}
</script>
