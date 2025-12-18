<template>
	<view class="content">
		<scroll-view scroll-y class="left-aside">
			<view
				v-for="item in flist"
				:key="item.id"
				class="f-item b-b"
				:class="{active: item.id === currentId}"
				@click="tabtap(item)">
				{{item.name}}
			</view>
		</scroll-view>

		<scroll-view scroll-with-animation scroll-y class="right-aside">
			<view class="s-list">
				<view
					@click="navToList(item.id)"
					class="s-item"
					v-for="item in slist"
					:key="item.id">
					<image :src="item.icon || 'http://macro-oss.oss-cn-shenzhen.aliyuncs.com/mall/images/20190519/default.png'"></image>
					<text>{{item.name}}</text>
				</view>
			</view>
		</scroll-view>
	</view>
</template>

<script>
	import { fetchProductCateList } from '@/api/home.js';

	export default {
		data() {
			return {
				currentId: 0,
				flist: [],
				slist: []
			}
		},
		onLoad() {
			this.loadData();
		},
		methods: {
			async loadData() {
				fetchProductCateList(0).then(response => {
					this.flist = response.data;
					if (this.flist.length > 0) {
						this.currentId = this.flist[0].id;
						fetchProductCateList(this.currentId).then(response => {
							this.slist = response.data;
						});
					}
				})
			},
			// Primary category click
			tabtap(item) {
				this.currentId = item.id;
				fetchProductCateList(this.currentId).then(response => {
					this.slist = response.data;
				});
			},
			navToList(sid) {
				uni.navigateTo({
					url: `/pages/product/list?fid=${this.currentId}&sid=${sid}`
				})
			}
		}
	}
</script>
