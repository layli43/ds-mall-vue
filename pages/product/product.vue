<template>
	<view class="container">
		<view class="carousel">
			<swiper indicator-dots circular=true duration="400">
				<swiper-item class="swiper-item" v-for="(item,index) in imgList" :key="index">
					<view class="image-wrapper">
						<image :src="item.src" class="loaded" mode="aspectFill"></image>
					</view>
				</swiper-item>
			</swiper>
		</view>

		<view class="introduce-section">
			<text class="title">{{product.name}}</text><br>
			<text class="title2">{{product.subTitle}}</text>
			<view class="price-box">
				<text class="price-tip">¥</text>
				<text class="price">{{product.price}}</text>
				<text class="m-price">¥{{product.originalPrice}}</text>
			</view>
			<view class="bot-row">
				<text>Sales: {{product.sale}}</text>
				<text>Stock: {{product.stock}}</text>
				<text>Views: 768</text>
			</view>
		</view>

		<!-- Share -->
		<view class="share-section" @click="share">
			<view class="share-icon">
				<text class="yticon icon-xingxing"></text>
				Ret
			</view>
			<text class="tit">Share to get a 49-10 red packet</text>
			<text class="yticon icon-bangzhu1"></text>
			<view class="share-btn">
				Share Now
				<text class="yticon icon-you"></text>
			</view>
		</view>

		<view class="c-list">
			<view class="c-row b-b" @click="toggleSpec">
				<text class="tit">Purchase Type</text>
				<view class="con">
					<text class="selected-text" v-for="(sItem, sIndex) in specSelected" :key="sIndex">
						{{sItem.name}}
					</text>
				</view>
				<text class="yticon icon-you"></text>
			</view>

			<view class="c-row b-b" @click="toggleAttr">
				<text class="tit">Product Parameters</text>
				<view class="con">
					<text class="con t-r">View</text>
				</view>
				<text class="yticon icon-you"></text>
			</view>

			<view class="c-row b-b" @click="toggleCoupon('show')">
				<text class="tit">Coupon</text>
				<text class="con t-r red">Get Coupon</text>
				<text class="yticon icon-you"></text>
			</view>

			<view class="c-row b-b">
				<text class="tit">Promotions</text>
				<view class="con-list">
					<text v-for="item in promotionTipList" :key="item">{{item}}</text>
				</view>
			</view>

			<view class="c-row b-b">
				<text class="tit">Services</text>
				<view class="bz-list con">
					<text v-for="item in serviceList" :key="item">{{item}} ·</text>
				</view>
			</view>
		</view>

		<!-- Reviews -->
		<view class="eva-section">
			<view class="e-header">
				<text class="tit">Reviews</text>
				<text>(86)</text>
				<text class="tip">100% Positive</text>
				<text class="yticon icon-you"></text>
			</view>
			<view class="eva-box">
				<image class="portrait" src="http://img3.imgtn.bdimg.com/it/u=1150341365,1327279810&fm=26&gp=0.jpg" mode="aspectFill"></image>
				<view class="right">
					<text class="name">Leo yo</text>
					<text class="con">
						Product received. 79 yuan for two items, good quality.
						A bit tight, but looks great with a jacket. I like it.
					</text>
					<view class="bot">
						<text class="attr">Purchase Type: XL Red</text>
						<text class="time">2019-04-01 19:21</text>
					</view>
				</view>
			</view>
		</view>

		<!-- Brand Info -->
		<view class="brand-info">
			<view class="d-header">
				<text>Brand Information</text>
			</view>
			<view class="brand-box" @click="navToBrandDetail()">
				<view class="image-wrapper">
					<image :src="brand.logo" class="loaded" mode="aspectFit"></image>
				</view>
				<view class="title">
					<text>{{brand.name}}</text>
					<text>Brand Initial: {{brand.firstLetter}}</text>
				</view>
			</view>
		</view>

		<view class="detail-desc">
			<view class="d-header">
				<text>Details</text>
			</view>
			<rich-text :nodes="desc"></rich-text>
		</view>

		<!-- Bottom Action Menu -->
		<view class="page-bottom">
			<navigator url="/pages/index/index" open-type="switchTab" class="p-b-btn">
				<text class="yticon icon-xiatubiao--copy"></text>
				<text>Home</text>
			</navigator>

			<navigator url="/pages/cart/cart" open-type="switchTab" class="p-b-btn">
				<text class="yticon icon-gouwuche"></text>
				<text>Cart</text>
			</navigator>

			<view class="p-b-btn" :class="{active: favorite}" @click="toFavorite">
				<text class="yticon icon-shoucang"></text>
				<text>Favorite</text>
			</view>

			<view class="action-btn-group">
				<button type="primary" class="action-btn no-border buy-now-btn" @click="buy">
					Buy Now
				</button>
				<button type="primary" class="action-btn no-border add-cart-btn" @click="addToCart">
					Add to Cart
				</button>
			</view>
		</view>

		<!-- Specification Modal -->
		<view class="popup spec" :class="specClass" @touchmove.stop.prevent="stopPrevent" @click="toggleSpec">
			<!-- Mask -->
			<view class="mask"></view>
			<view class="layer attr-content" @click.stop="stopPrevent">
				<view class="a-t">
					<image :src="product.pic"></image>
					<view class="right">
						<text class="price">¥{{product.price}}</text>
						<text class="stock">Stock: {{product.stock}} items</text>
						<view class="selected">
							Selected:
							<text class="selected-text" v-for="(sItem, sIndex) in specSelected" :key="sIndex">
								{{sItem.name}}
							</text>
						</view>
					</view>
				</view>
				<view v-for="(item,index) in specList" :key="index" class="attr-list">
					<text>{{item.name}}</text>
					<view class="item-list">
						<text
							v-for="(childItem, childIndex) in specChildList"
							v-if="childItem.pid === item.id"
							:key="childIndex"
							class="tit"
							:class="{selected: childItem.selected}"
							@click="selectSpec(childIndex, childItem.pid)">
							{{childItem.name}}
						</text>
					</view>
				</view>
				<button class="btn" @click="toggleSpec">Done</button>
			</view>
		</view>

		<!-- Attribute Modal -->
		<view class="popup spec" :class="attrClass" @touchmove.stop.prevent="stopPrevent" @click="toggleAttr">
			<!-- Mask -->
			<view class="mask"></view>
			<view class="layer attr-content no-padding" @click.stop="stopPrevent">
				<view class="c-list">
					<view v-for="item in attrList" class="c-row b-b" :key="item.key">
						<text class="tit">{{item.key}}</text>
						<view class="con">
							<text class="con t-r">{{item.value}}</text>
						</view>
					</view>
				</view>
			</view>
		</view>

		<!-- Coupon Panel -->
		<view class="mask" :class="couponState===0 ? 'none' : couponState===1 ? 'show' : ''" @click="toggleCoupon">
			<view class="mask-content" @click.stop.prevent="stopPrevent">
				<!-- Coupon page (Meituan-style) -->
				<view class="coupon-item" v-for="(item,index) in couponList" :key="index" @click="addCoupon(item)">
					<view class="con">
						<view class="left">
							<text class="title">{{item.name}}</text>
							<text class="time">Valid until {{item.endTime | formatDateTime}}</text>
						</view>
						<view class="right">
							<text class="price">{{item.amount}}</text>
							<text>Available for orders over {{item.minPoint}}</text>
						</view>

						<view class="circle l"></view>
						<view class="circle r"></view>
					</view>
					<text class="tips">{{item.useType | formatCouponUseType}}</text>
				</view>
			</view>
		</view>

		<!-- Share -->
		<share ref="share" :contentHeight="580" :shareList="shareList"></share>
	</view>
</template>
