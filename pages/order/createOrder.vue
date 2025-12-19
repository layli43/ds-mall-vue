<template>
	<view>
		<!-- Address -->
		<navigator url="/pages/address/address?source=1" class="address-section">
			<view class="order-content">
				<text class="yticon icon-shouhuodizhi"></text>
				<view class="cen">
					<view class="top">
						<text class="name">{{currentAddress.name}}</text>
						<text class="mobile">{{currentAddress.phoneNumber}}</text>
					</view>
					<text class="address">
						{{currentAddress.province}} {{currentAddress.city}} {{currentAddress.region}}
						{{currentAddress.detailAddress}}
					</text>
				</view>
				<text class="yticon icon-you"></text>
			</view>

			<image class="a-bg" src="data:image/png;base64,..."></image>
		</navigator>

		<view class="goods-section">
			<view class="g-header b-b">
				<text class="name">Product Information</text>
			</view>
			<!-- Product list -->
			<view class="g-item" v-for="item in cartPromotionItemList" :key="item.id">
				<image :src="item.productPic"></image>
				<view class="right">
					<text class="title clamp">{{item.productName}}</text>
					<text class="spec">{{item.productAttr | formatProductAttr}}</text>
					<text class="promotion clamp">{{item.promotionMessage}}</text>
					<view class="price-box">
						<text class="price">￥{{item.price}}</text>
						<text class="number">x {{item.quantity}}</text>
					</view>
				</view>
			</view>
		</view>

		<!-- Discount Details -->
		<view class="yt-list">
			<view class="yt-list-cell b-b" @click="toggleMask('show')">
				<view class="cell-icon">Coupon</view>
				<text class="cell-tit clamp">Coupon</text>
				<text class="cell-tip active">Select Coupon</text>
				<text class="cell-more wanjia wanjia-gengduo-d"></text>
			</view>

			<view class="yt-list-cell b-b">
				<view class="cell-icon hb">Pts</view>
				<text class="cell-tit clamp">Points Deduction</text>
				<input
					class="integration"
					type="number"
					v-model="useIntegration"
					placeholder="Points to use"
					placeholder-class="placeholder"
					@input="handleIntegrationInput"
				/>
			</view>
		</view>

		<!-- Amount Details -->
		<view class="yt-list">
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Subtotal</text>
				<text class="cell-tip">￥{{calcAmount.totalAmount}}</text>
			</view>
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Shipping</text>
				<text class="cell-tip">￥{{calcAmount.freightAmount}}</text>
			</view>
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Promotion Discount</text>
				<text class="cell-tip red">-￥{{calcAmount.promotionAmount}}</text>
			</view>
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Coupon</text>
				<text class="cell-tip red" v-if="currCoupon!=null">-￥{{currCoupon.amount}}</text>
				<text class="cell-tip red" v-else>-￥0</text>
			</view>
			<view class="yt-list-cell b-b">
				<text class="cell-tit clamp">Points Deduction</text>
				<text class="cell-tip red">-￥{{calcIntegrationAmount(useIntegration)}}</text>
			</view>
			<view class="yt-list-cell desc-cell">
				<text class="cell-tit clamp">Notes</text>
				<input
					class="desc"
					type="text"
					v-model="desc"
					placeholder="Please enter notes"
					placeholder-class="placeholder"
				/>
			</view>
		</view>

		<!-- Footer -->
		<view class="footer">
			<view class="price-content">
				<text>Amount to Pay</text>
				<text class="price-tip">￥</text>
				<text class="price">{{calcAmount.payAmount}}</text>
			</view>
			<text class="submit" @click="submit">Submit Order</text>
		</view>

		<!-- Coupon Panel -->
		<view class="mask" :class="maskState===0 ? 'none' : maskState===1 ? 'show' : ''" @click="toggleMask">
			<view class="mask-content" @click.stop.prevent="stopPrevent">
				<!-- Coupon page (Meituan-style) -->
				<view
					class="coupon-item"
					v-for="(item,index) in couponList"
					:key="index"
					@click="selectCoupon(item)"
				>
					<view class="con">
						<view class="left">
							<text class="title">{{item.name}}</text>
							<text class="time">
								Valid until {{item.endTime | formatDateTime}}
							</text>
						</view>
						<view class="right">
							<text class="price">{{item.amount}}</text>
							<text>Usable on orders over {{item.minPoint}}</text>
						</view>

						<view class="circle l"></view>
						<view class="circle r"></view>
					</view>
					<text class="tips">{{item.useType | formatCouponUseType}}</text>
				</view>
			</view>
		</view>

	</view>
</template>

<script>
export default {
	data() {
		return {
			maskState: 0, // Coupon panel visibility state
			desc: '', // Notes
			payType: 1, // 1 = WeChat Pay, 2 = Alipay
			// ...
		}
	},
	filters: {
		formatCouponUseType(useType) {
			if (useType == 0) {
				return "Universal";
			} else if (useType == 1) {
				return "Applicable to specific categories";
			} else if (useType == 2) {
				return "Applicable to specific products";
			}
			return null;
		},
	},
	methods: {
		// Generate order confirmation data
		loadData() {},

		// Show coupon panel
		toggleMask(type) {},

		// Get default shipping address
		getDefaultAddress() {},

		// Calculate payable amount
		calcPayAmount() {},

		// Convert points to amount
		calcIntegrationAmount() {},
	}
}
</script>
