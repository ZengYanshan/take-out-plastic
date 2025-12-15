<template>
	<view class="shop-detail">
		<view class="status-bar" style="height: var(--status-bar-height); width: 100%;" />
<!-- 		<view class="custom-navbar">
			<image src="/static/images/icon-back.png" class="back-icon" @click="goBack" />
			<text class="page-title">{{ shop.name }}</text>
		</view> -->
		<view class="main-container">
			<shop-header :shop="shop" />

			<!-- <view class="tab-wrapper">
				<tab :tabs="tabs" />
			</view> -->
			
			<view class="food-list-wrapper">
				<!-- <food-list :data="{shop: shop}"/> -->
				<food-list ref="foodList" :shop="shop" />
			</view>
		</view>
	</view>
</template>

<script>
	import ShopHeader from '@/components/ShopHeader.vue';
	// import Tab from '@/components/Tab.vue';
	import FoodList from '@/components/FoodList.vue';
	import CommentList from '@/components/CommentList.vue';
	import SHOPS from '@/static/data/shops.js';
	import FOODS from '@/static/data/foods.js';

	export default {
		components: {
			ShopHeader,
			FoodList
			// Tab
		},
		data() {
			return {
				shop: {}
			};
		},
		computed: {
			tabs() {
				if (this.shop) {
					console.log("shop数据已加载，可传正确tab");
				}
				
				return [{
						label: '商品',
						component: FoodList,
						data: {
							shop: this.shop
						}
					},
					{
						label: '评论',
						component: CommentList,
						data: {
							shop: this.shop
						}
					}
				]
			},
		},
		onLoad(options) {
			this.shop = SHOPS.find(s => s.id == options.shopId) || {};
			if (options.foodId) {
				let parentSelectedFood = FOODS.find(f => f.id == options.foodId) || {};
				this.$nextTick(() => {
					this.$refs.foodList.showFoodDetail(parentSelectedFood);
				});
			}
		},
		methods: {
			goBack() {
				uni.navigateBack();
			}
		}
	};
</script>

<style scoped>
	.shop-detail {
		position: fixed;
		width: 100%;
		display: flex;
		flex-direction: column;
		height: 100vh;
		background: #fff;
	}

	.custom-navbar {
		display: flex;
		align-items: center;
		padding: 15rpx 30rpx;
		background-color: #f7931e;
		color: white;
	}

	.back-icon,
	.action-icon {
		width: 44rpx;
		height: 44rpx;
	}

	.main-container {
		display: flex;
		flex-direction: column;
		flex: 1;
		overflow: hidden;
	}
	
	.food-list-wrapper {
		flex: 1;
		height: calc(100vh - 178px);
		/* position: fixed; */
		/* top: 256rpx; */
		/* left: 0;
		right: 0;
		bottom: 0; */
		/* background: white; */
	}
</style>