<template>
	<view class="container">
		<!-- Empty Page -->
		<view v-if="!hasLogin || empty===true" class="empty">
			<image src="/static/emptyCart.jpg" mode="aspectFit"></image>
			<view v-if="hasLogin" class="empty-tips">
				Empty cart
				<navigator class="navigator" v-if="hasLogin" url="../index/index" open-type="switchTab">
					Browse >
				</navigator>
			</view>
			<view v-else class="empty-tips">
				Empty cart
				<view class="navigator" @click="navToLogin">Login ></view>
			</view>
		</view>

		<view v-else>
			<!-- Cart List -->
			<view class="cart-list">
				<block v-for="(item, index) in cartList" :key="item.id">
					<view class="cart-item" :class="{'b-b': index!==cartList.length-1}">
						<view class="image-wrapper">
							<image
								:src="item.productPic"
								:class="[item.loaded]"
								mode="aspectFill"
								lazy-load
								@load="onImageLoad('cartList', index)"
								@error="onImageError('cartList', index)">
							</image>
							<view
								class="yticon icon-xuanzhong2 checkbox"
								:class="{checked: item.checked}"
								@click="check('item', index)">
							</view>
						</view>

						<view class="item-right">
							<text class="clamp title">{{item.productName}}</text>
							<text class="attr">{{item.spDataStr}}</text>
							<text class="price">¥{{item.price}}</text>
							<uni-number-box
								class="step"
								:min="1"
								:max="100"
								:value="item.quantity"
								:index="index"
								@eventChange="numberChange">
							</uni-number-box>
						</view>

						<text class="del-btn yticon icon-fork" @click="handleDeleteCartItem(index)"></text>
					</view>
				</block>
			</view>

			<!-- Bottom Action Bar -->
			<view class="action-section">
				<view class="checkbox">
					<image
						:src="allChecked?'/static/selected.png':'/static/select.png'"
						mode="aspectFit"
						@click="check('all')">
					</image>
					<view class="clear-btn" :class="{show: allChecked}" @click="clearCart">
						Clear
					</view>
				</view>

				<view class="total-box">
					<text class="price">¥{{total}}</text>
				</view>

				<button
					type="primary"
					class="no-border confirm-btn"
					@click="createOrder">
					Checkout
				</button>
			</view>
		</view>
	</view>
</template>

<script>
	import { mapState } from 'vuex';
	import uniNumberBox from '@/components/uni-number-box.vue';
	import {
		fetchCartList,
		deletCartItem,
		updateQuantity,
		clearCartList
	} from '@/api/cart.js';

	export default {
		components: {
			uniNumberBox
		},
		data() {
			return {
				total: 0,          // Total price
				allChecked: false, // Select all status: true | false
				empty: false,      // Show empty page: true | false
				cartList: [],
			};
		},
		onLoad() {
			// this.loadData();
		},
		onShow() {
			// Reload the cart when the page is displayed
			this.loadData();
		},
		watch: {
			// Show empty page when cart is empty
			cartList(e) {
				let empty = e.length === 0;
				if (this.empty !== empty) {
					this.empty = empty;
				}
			}
		},
		computed: {
			...mapState(['hasLogin'])
		},
		methods: {
			// Request cart data
			async loadData() {
				if (!this.hasLogin) {
					return;
				}
				fetchCartList().then(response => {
					let list = response.data;
					let cartList = list.map(item => {
						item.checked = true;
						item.loaded = "loaded";
						let spDataArr = JSON.parse(item.productAttr);
						let spDataStr = '';
						for (let attr of spDataArr) {
							spDataStr += attr.key + ":" + attr.value + ";";
						}
						item.spDataStr = spDataStr;
						return item;
					});
					this.cartList = cartList;
					this.calcTotal(); // Calculate total price
				});
			},

			// Image load success handler
			onImageLoad(key, index) {
				this.$set(this[key][index], 'loaded', 'loaded');
			},

			// Image load error handler
			onImageError(key, index) {
				this[key][index].productPic = '/static/errorImage.jpg';
			},

			// Navigate to login page
			navToLogin() {
				uni.navigateTo({
					url: '/pages/public/login'
				});
			},

			// Handle item selection
			check(type, index) {
				if (type === 'item') {
					this.cartList[index].checked = !this.cartList[index].checked;
				} else {
					const checked = !this.allChecked;
					this.cartList.forEach(item => {
						item.checked = checked;
					});
					this.allChecked = checked;
				}
				this.calcTotal();
			},

			// Quantity change handler
			numberChange(data) {
				let cartItem = this.cartList[data.index];
				updateQuantity({ id: cartItem.id, quantity: data.number }).then(() => {
					cartItem.quantity = data.number;
					this.calcTotal();
				});
			},

			// Delete cart item
			handleDeleteCartItem(index) {
				let row = this.cartList[index];
				deletCartItem({ ids: row.id }).then(() => {
					this.cartList.splice(index, 1);
					this.calcTotal();
					uni.hideLoading();
				});
			},

			// Clear cart
			clearCart() {
				clearCartList().then(() => {
					uni.showModal({
						content: 'Clear shopping cart?',
						success: (e) => {
							if (e.confirm) {
								this.cartList = [];
							}
						}
					});
				});
			},

			// Calculate total price
			calcTotal() {
				let list = this.cartList;
				if (list.length === 0) {
					this.empty = true;
					return;
				}
				let total = 0;
				let checked = true;
				list.forEach(item => {
					if (item.checked) {
						total += item.price * item.quantity;
					} else {
						checked = false;
					}
				});
				this.allChecked = checked;
				this.total = Number(total.toFixed(2));
			},

			// Create order
			createOrder() {
				let cartIds = [];
				this.cartList.forEach(item => {
					if (item.checked) {
						cartIds.push(item.id);
					}
				});
				if (cartIds.length === 0) {
					uni.showToast({
						title: "You haven't selected any products to order!",
						duration: 1000
					});
					return;
				}
				uni.navigateTo({
					url: `/pages/order/createOrder?cartIds=${JSON.stringify(cartIds)}`
				});
			}
		}
	};
</script>
