<template>
	<view>
		<view class="status-section">
			<image :src="orderStatus.image" class="icon" />
			<text class="label-text">{{orderStatus.text}}</text>
		</view>

		<!-- Address -->
		<view class="address-section">
			<view class="order-content">
				<text class="yticon icon-shouhuodizhi"></text>
				<view class="cen">
					<view class="top">
						<text class="name">{{order.receiverName}}</text>
						<text class="mobile">{{order.receiverPhone}}</text>
					</view>
					<text class="address">
						{{order.receiverProvince}} {{order.receiverCity}} {{order.receiverRegion}}
						{{order.receiverDetailAddress}}
					</text>
				</view>
			</view>

			<image class="a-bg" src="data:image/png;base64,..."></image>
		</view>

		<view class="goods-section">
			<view class="g-header b-b">
				<text class="name">Product Information</text>
			</view>
			<!-- Product list -->
			<view class="g-item" v-for="item in order.orderItemList" :key="item.id">
				<image :src="item.productPic"></image>
				<view class="right">
					<text class="title clamp">{{item.productName}}</text>
					<text class="spec">{{item.productAttr | formatProductAttr}}</text>
					<text class="promotion clamp">{{item.promotionName}}</text>
					<view class="price-box">
						<text class="price">￥{{item.productPrice}}</text>
						<text class="number">x {{item.productQuantity}}</text>
					</view>
				</view>
			</view>
		</view>

		<!-- Amount Details -->
		<view class="yt-list">
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Items Subtotal</text>
				<text class="cell-tip">￥{{order.totalAmount}}</text>
			</view>
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Shipping Fee</text>
				<text class="cell-tip">￥{{order.freightAmount}}</text>
			</view>
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Promotion Discount</text>
				<text class="cell-tip red">-￥{{order.promotionAmount}}</text>
			</view>
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Coupon</text>
				<text class="cell-tip red">-￥{{order.couponAmount}}</text>
			</view>
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Points Deduction</text>
				<text class="cell-tip red">-￥{{order.integrationAmount}}</text>
			</view>
			<view class="yt-list-cell desc-cell">
				<text class="cell-tit clamp">Notes</text>
				<text class="cell-tip">{{order.note}}</text>
			</view>
		</view>

		<!-- Order Details -->
		<view class="yt-list">
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Order Number</text>
				<text class="cell-tip">{{order.orderSn}}</text>
			</view>
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Submitted At</text>
				<text class="cell-tip">{{order.createTime | formatDateTime}}</text>
			</view>
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Payment Method</text>
				<text class="cell-tip">{{order.payType | formatPayType}}</text>
			</view>
			<view class="yt-list-cell b-b" v-if="order.status==1||order.status==2||order.status==3">
				<text class="cell-tit clamp">Amount Paid</text>
				<text class="cell-tip">￥{{order.payAmount}}</text>
			</view>
			<view class="yt-list-cell b-b" v-if="order.status==1||order.status==2||order.status==3">
				<text class="cell-tit clamp">Payment Time</text>
				<text class="cell-tip">{{order.paymentTime | formatDateTime}}</text>
			</view>
		</view>

		<!-- Footer -->
		<view class="footer" v-if="order.status==0||order.status==2||order.status==3">
			<view class="action-box b-t" v-if="order.status==0">
				<button class="action-btn" @click="cancelOrder(order.id)">Cancel Order</button>
				<button class="action-btn recom" @click="payOrder(order.id)">Pay Now</button>
			</view>
			<view class="action-box b-t" v-if="order.status == 2">
				<button class="action-btn">Track Shipment</button>
				<button class="action-btn recom" @click="receiveOrder(order.id)">Confirm Receipt</button>
			</view>
			<view class="action-box b-t" v-if="order.status == 3">
				<button class="action-btn">Request After-sales</button>
				<button class="action-btn recom">Rate Product</button>
			</view>
			<view class="price-content" v-if="order.status==0">
				<text>Amount Due</text>
				<text class="price-tip">￥</text>
				<text class="price">{{order.payAmount}}</text>
			</view>
		</view>

	</view>
</template>

<script>
import { fetchOrderDetail, cancelUserOrder, confirmReceiveOrder } from '@/api/order.js';
import { formatDate } from '@/utils/date';

export default {
	data() {
		return {
			orderId: null,
			order: {},
			orderStatus: {}
		}
	},
	onLoad(option) {
		// Product data
		this.orderId = option.orderId;
		this.loadData();
	},
	filters: {
		formatProductAttr(jsonAttr) {
			let attrArr = JSON.parse(jsonAttr);
			let attrStr = '';
			for (let attr of attrArr) {
				attrStr += attr.key + ":" + attr.value + ";";
			}
			return attrStr;
		},
		formatDateTime(time) {
			if (time == null || time === '') return 'N/A';
			let date = new Date(time);
			return formatDate(date, 'yyyy-MM-dd hh:mm:ss');
		},
		formatPayType(payType) {
			if (payType == 0) return "Unpaid";
			else if (payType == 1) return "Alipay";
			else if (payType == 2) return "WeChat Pay";
			return null;
		},
	},
	methods: {
		// Load order details
		async loadData() {
			fetchOrderDetail(this.orderId).then(response => {
				this.order = response.data;
				this.setOrderStatus(this.order.status);
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
			uni.redirectTo({ url: `/pages/money/pay?orderId=${orderId}` });
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
		// Set order status display
		setOrderStatus(status) {
			switch (status) {
				case 0:
					this.orderStatus = { text: 'Awaiting Payment', image: '/static/icon_wait.png' };
					break;
				case 1:
					this.orderStatus = { text: 'Awaiting Shipment', image: '/static/icon_deliver.png' };
					break;
				case 2:
					this.orderStatus = { text: 'Awaiting Receipt', image: '/static/icon_receive.png' };
					break;
				case 3:
					this.orderStatus = { text: 'Completed', image: '/static/icon_finish.png' };
					break;
				case 4:
					this.orderStatus = { text: 'Closed', image: '/static/icon_close.png' };
					break;
			}
		}
	}
}
</script>
