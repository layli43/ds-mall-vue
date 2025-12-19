<template>
	<view class="content">
		<view class="navbar">
			<view
				v-for="(item, index) in navList"
				:key="index"
				class="nav-item"
				:class="{current: tabCurrentIndex === index}"
				@click="tabClick(index)"
			>
				{{item.text}}
			</view>
		</view>

		<swiper
			:current="tabCurrentIndex"
			class="swiper-box"
			duration="300"
			@change="changeTab"
		>
			<swiper-item
				class="tab-content"
				v-for="(tabItem,tabIndex) in navList"
				:key="tabIndex"
			>
				<scroll-view
					class="list-scroll-content"
					scroll-y
					@scrolltolower="loadData('add')"
				>
					<!-- Empty page -->
					<empty v-if="orderList==null||orderList.length === 0"></empty>

					<!-- Order list -->
					<view
						v-for="(item,index) in orderList"
						:key="index"
						class="order-item"
					>
						<view class="i-top b-b">
							<text class="time" @click="showOrderDetail(item.id)">
								{{item.createTime | formatDateTime}}
							</text>
							<text class="state" :style="{color: '#fa436a'}">
								{{item.status | formatStatus}}
							</text>
							<text
								v-if="item.status===3||item.status===4"
								class="del-btn yticon icon-iconfontshanchu1"
								@click="deleteOrder(item.id)"
							></text>
						</view>

						<view
							class="goods-box-single"
							v-for="(orderItem, itemIndex) in item.orderItemList"
							:key="itemIndex"
						>
							<image
								class="goods-img"
								:src="orderItem.productPic"
								mode="aspectFill"
							></image>
							<view class="right">
								<text class="title clamp">
									{{orderItem.productName}}
								</text>
								<text class="attr-box">
									{{orderItem.productAttr | formatProductAttr}}
									x {{orderItem.productQuantity}}
								</text>
								<text class="price">
									{{orderItem.productPrice}}
								</text>
							</view>
						</view>

						<view class="price-box">
							Total
							<text class="num">{{calcTotalQuantity(item)}}</text>
							items, Amount Paid
							<text class="price">{{item.payAmount}}</text>
						</view>

						<view class="action-box b-t" v-if="item.status == 0">
							<button class="action-btn" @click="cancelOrder(item.id)">
								Cancel Order
							</button>
							<button class="action-btn recom" @click="payOrder(item.id)">
								Pay Now
							</button>
						</view>

						<view class="action-box b-t" v-if="item.status == 2">
							<button class="action-btn">Track Shipment</button>
							<button class="action-btn recom" @click="receiveOrder(item.id)">
								Confirm Receipt
							</button>
						</view>

						<view class="action-box b-t" v-if="item.status == 3">
							<button class="action-btn recom">Rate Product</button>
						</view>
					</view>

					<uni-load-more :status="loadingType"></uni-load-more>
				</scroll-view>
			</swiper-item>
		</swiper>
	</view>
</template>

<script>
import uniLoadMore from '@/components/uni-load-more/uni-load-more.vue';
import empty from "@/components/empty";
import { formatDate } from '@/utils/date';
import {
	fetchOrderList,
	cancelUserOrder,
	confirmReceiveOrder,
	deleteUserOrder
} from '@/api/order.js';

export default {
	components: {
		uniLoadMore,
		empty
	},
	data() {
		return {
			tabCurrentIndex: 0,
			orderParam: {
				status: -1,
				pageNum: 1,
				pageSize: 5
			},
			orderList: [],
			loadingType: 'more',
			navList: [
				{ state: -1, text: 'All' },
				{ state: 0, text: 'Pending Payment' },
				{ state: 2, text: 'To Be Received' },
				{ state: 3, text: 'Completed' },
				{ state: 4, text: 'Cancelled' }
			],
		};
	},
	onLoad(options) {
		/**
		 * Fix the issue where data is not loaded
		 * when entering the page by clicking tabs other than "All" on app side
		 * Replace the code under onLoad with this
		 */
		this.tabCurrentIndex = +options.state;

		// #ifndef MP
		this.loadData();
		// #endif

		// #ifdef MP
		if (options.state == 0) {
			this.loadData();
		}
		// #endif
	},
	filters: {
		formatStatus(status) {
			let statusTip = '';
			switch (+status) {
				case 0:
					statusTip = 'Awaiting Payment';
					break;
				case 1:
					statusTip = 'Awaiting Shipment';
					break;
				case 2:
					statusTip = 'Awaiting Receipt';
					break;
				case 3:
					statusTip = 'Completed';
					break;
				case 4:
					statusTip = 'Closed';
					break;
			}
			return statusTip;
		},
		formatProductAttr(jsonAttr) {
			let attrArr = JSON.parse(jsonAttr);
			let attrStr = '';
			for (let attr of attrArr) {
				attrStr += attr.key + ":" + attr.value + ";";
			}
			return attrStr;
		},
		formatDateTime(time) {
			if (time == null || time === '') {
				return 'N/A';
			}
			let date = new Date(time);
			return formatDate(date, 'yyyy-MM-dd hh:mm:ss');
		},
	},
	methods: {
		// Get order list
		loadData(type = 'refresh') {
			if (type === 'refresh') {
				this.orderParam.pageNum = 1;
			} else {
				this.orderParam.pageNum++;
			}

			let navItem = this.navList[this.tabCurrentIndex];

			if (this.loadingType === 'loading') {
				// Prevent duplicate loading
				return;
			}

			this.orderParam.status = navItem.state;
			this.loadingType = 'loading';

			fetchOrderList(this.orderParam).then(response => {
				let list = response.data.list;
				if (type === 'refresh') {
					this.orderList = list;
					this.loadingType = 'more';
				} else {
					if (list && list.length > 0) {
						this.orderList = this.orderList.concat(list);
						this.loadingType = 'more';
					} else {
						this.orderParam.pageNum--;
						this.loadingType = 'noMore';
					}
				}
			});
		},

		// Swiper tab switch
		changeTab(e) {
			this.tabCurrentIndex = e.target.current;
			this.loadData();
		},

		// Top tab click
		tabClick(index) {
			this.tabCurrentIndex = index;
		},

		// Delete order
		deleteOrder(orderId) {
			let superThis = this;
			uni.showModal({
				title: 'Notice',
				content: 'Do you want to delete this order?',
				success(res) {
					if (res.confirm) {
						uni.showLoading({ title: 'Please wait' });
						deleteUserOrder({ orderId }).then(() => {
							uni.hideLoading();
							superThis.loadData();
						});
					}
				}
			});
		},

		// Cancel order
		cancelOrder(orderId) {
			let superThis = this;
			uni.showModal({
				title: 'Notice',
				content: 'Do you want to cancel this order?',
				success(res) {
					if (res.confirm) {
						uni.showLoading({ title: 'Please wait' });
						cancelUserOrder({ orderId }).then(() => {
							uni.hideLoading();
							superThis.loadData();
						});
					} else if (res.cancel) {
						console.log('User clicked cancel');
					}
				}
			});
		},

		// Pay order
		payOrder(orderId) {
			uni.redirectTo({
				url: `/pages/money/pay?orderId=${orderId}`
			});
		},

		// Confirm receipt
		receiveOrder(orderId) {
			let superThis = this;
			uni.showModal({
				title: 'Notice',
				content: 'Do you want to confirm receipt?',
				success(res) {
					if (res.confirm) {
						uni.showLoading({ title: 'Please wait' });
						confirmReceiveOrder({ orderId }).then(() => {
							uni.hideLoading();
							superThis.loadData();
						});
					} else if (res.cancel) {
						console.log('User clicked cancel');
					}
				}
			});
		},

		// View order details
		showOrderDetail(orderId) {
			uni.navigateTo({
				url: `/pages/order/orderDetail?orderId=${orderId}`
			});
		},

		// Calculate total product quantity
		calcTotalQuantity(order) {
			let totalQuantity = 0;
			if (order.orderItemList && order.orderItemList.length > 0) {
				for (let item of order.orderItemList) {
					totalQuantity += item.productQuantity;
				}
			}
			return totalQuantity;
		},
	},
};
</script>
