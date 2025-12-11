<template>
		<view class="content">
			<image src="/static/recommend_brand_banner.png" class="banner-image"></image>
			<view class="section-tit">Related Brands</view>
			<view class="goods-list">
				<view v-for="(item, index) in brandList" :key="index" class="goods-item" @click="navToDetailPage(item)">
					<view class="image-wrapper">
					<image :src="item.logo" mode="aspectFit"></image>
				</view>
				<text class="title clamp">{{item.name}}</text>
				<text class="title2">Product Count: {{item.productCount}}</text>
			</view>
			</view>
			<uni-load-more :status="loadingType"></uni-load-more>
		</view>
</template>

<script>
		import {
			fetchBrandRecommendList
		} from '@/api/brand.js';
		import uniLoadMore from '@/components/uni-load-more/uni-load-more.vue';
</script>

<style>
</style>