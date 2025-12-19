<template>
	<view class="container">
		<view class="list-cell b-b m-t" @click="navTo('Profile')" hover-class="cell-hover" :hover-stay-time="50">
			<text class="cell-tit">Profile</text>
			<text class="cell-more yticon icon-you"></text>
		</view>
		<view class="list-cell b-b" @click="navTo('/pages/address/address')" hover-class="cell-hover" :hover-stay-time="50">
			<text class="cell-tit">Shipping Address</text>
			<text class="cell-more yticon icon-you"></text>
		</view>
		<view class="list-cell" @click="navTo('ID Verification')" hover-class="cell-hover" :hover-stay-time="50">
			<text class="cell-tit">ID Verification</text>
			<text class="cell-more yticon icon-you"></text>
		</view>
		
		<view class="list-cell m-t">
			<text class="cell-tit">Push Notifications</text>
			<switch checked color="#fa436a" @change="switchChange" />
		</view>
		<view class="list-cell m-t b-b" @click="navTo('Clear Cache')" hover-class="cell-hover" :hover-stay-time="50">
			<text class="cell-tit">Clear Cache</text>
			<text class="cell-more yticon icon-you"></text>
		</view>
		<view class="list-cell b-b" @click="navToOuter('https://github.com/macrozheng/mall')" hover-class="cell-hover" :hover-stay-time="50">
			<text class="cell-tit">About mall-app-web</text>
			<text class="cell-more yticon icon-you"></text>
		</view>
		<view class="list-cell">
			<text class="cell-tit">Check for Updates</text>
			<text class="cell-tip">Current Version 1.0.0</text>
			<text class="cell-more yticon icon-you"></text>
		</view>
		<view class="list-cell log-out-btn" @click="toLogout">
			<text class="cell-tit">Logout</text>
		</view>
	</view>
</template>

<script>
	import {  
	    mapMutations  
	} from 'vuex';
	export default {
		data() {
			return {
				
			};
		},
		methods:{
			...mapMutations(['logout']),

			// Navigation handler
			navTo(url){
				if(url.indexOf("pages") != -1){
					uni.navigateTo({
						url: url
					});
				}
				// Showing toast for placeholder routes
				this.$api.msg(`Navigating to ${url}`);
			},
			// External link handler
			navToOuter(url){
				window.location.href = url;
			},
			// Logout logic
			toLogout(){
				uni.showModal({
				    content: 'Are you sure you want to logout?',
				    success: (e)=>{
				    	if(e.confirm){
				    		this.logout();
				    		setTimeout(()=>{
				    			uni.navigateBack();
				    		}, 200)
				    	}
				    }
				});
			},
			// Notification switch handler
			switchChange(e){
				let statusTip = e.detail.value ? 'Enabled': 'Disabled';
				this.$api.msg(`Push notifications ${statusTip}`);
			},

		}
	}
</script>

<style lang='scss'>
	/* Styles remain unchanged as they use CSS properties */
	page{
		background: $page-color-base;
	}
	.list-cell{
		display:flex;
		align-items:baseline;
		padding: 20upx $page-row-spacing;
		line-height:60upx;
		position:relative;
		background: #fff;
		justify-content: center;
		&.log-out-btn{
			margin-top: 40upx;
			.cell-tit{
				color: $uni-color-primary;
				text-align: center;
				margin-right: 0;
			}
		}
		&.cell-hover{
			background:#fafafa;
		}
		&.b-b:after{
			left: 30upx;
		}
		&.m-t{
			margin-top: 16upx; 
		}
		.cell-more{
			align-self: baseline;
			font-size:$font-lg;
			color:$font-color-light;
			margin-left:10upx;
		}
		.cell-tit{
			flex: 1;
			font-size: $font-base + 2upx;
			color: $font-color-dark;
			margin-right:10upx;
		}
		.cell-tip{
			font-size: $font-base;
			color: $font-color-light;
		}
		switch{
			transform: translateX(16upx) scale(.84);
		}
	}
</style>