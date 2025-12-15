<template>
	<view class="shop-card">
		<view class="shop-info-wrapper" @click="onClickShop">
			<image class="shop-image" :src="shop.image" mode="aspectFill" />
			<view class="shop-info">
				<view class="shop-header">
					<text class="shop-name">{{ shop.name }}</text>
					<view class="shop-rating">
						<text class="rating-number">{{ shop.rating.toFixed(1) }}</text>
						<uni-rate :value="shop.rating / 5 * 5" size="12" readonly />
					</view>
				</view>

				<view class="shop-meta">
					<text class="delivery-info">
						{{ shop.deliveryTime }}外包装
					</text>
					<text class="delivery-fee">
						{{ shop.deliveryFee === 0 ? '免包装' : `包装降解需${shop.deliveryFee}年` }}
					</text>
				</view>

				<view class="shop-tags">
					<text v-for="(tag, index) in shop.tags" :key="index" class="tag">
						{{ tag }}
					</text>
				</view>

			</view>
		</view>

		<scroll-view class="signature-foods-wrapper" scroll-x enable-flex enhanced="true" show-scrollbar="{{false}}">
			<view v-for="signatureFood in signatureFoods" @click="onClickFood(signatureFood)" class="food-card">
				<view class="food-image-wrapper">
					<image :src="signatureFood.image" class="food-image" mode="aspectFill" />
				</view>
				<view class="food-name ellipsis">
					{{ signatureFood.name }}
				</view>
				<view class="food-price">
					{{ signatureFood.price }}年
				</view>
			</view>

			<view class="enter-shop-button" @click="onClickShop">
				进入商店
			</view>
		</scroll-view>
	</view>
</template>

<script>
	import FOODS from '@/static/data/foods.js';

	export default {
		name: 'ShopCard',
		data() {
			return {
				signatureFoods: [],
			};
		},
		props: {
			shop: {
				type: Object,
				required: true
			}
		},
		mounted() {
			this.signatureFoods = this.getSignatureFoods(this.shop);
			console.log("signatureFoods = ", this.signatureFoods);
		},
		methods: {
			onClickShop() {
				let options = {
					shopId: this.shop.id
				};
				this.$emit('click-shop', options);
				console.log('click-shop', options);
			},
			onClickFood(food) {
				let options = {
					shopId: this.shop.id,
					foodId: food.id
				};
				this.$emit('click-food', options);
				console.log('click-food', options);
			},
			getSignatureFoods(shop) {
				const signatureFoodIds = this.getSignatureFoodIds(shop);
				const signatureFoodIdsSet = new Set(signatureFoodIds);

				// 筛选
				return FOODS.filter(food => signatureFoodIdsSet.has(food.id));
			},
			getSignatureFoodIds(shop, count = 4) {
				return shop.menu.reduce((acc, category) => {
					// 如果已经收集了足够的数量，停止处理
					if (acc.length >= count) return acc;

					// 遍历当前分类的items
					for (const item of category.items) {
						if (acc.length < count) {
							acc.push(item.foodId);
						} else {
							break; // 已经收集了足够的数量，跳出循环
						}
					}

					return acc;
				}, []);
			}
		}
	};
</script>

<style scoped>
	.shop-card {
		margin: 24rpx 20rpx;
		border-radius: 10rpx;
		
		background-color: #ffffff;
		
		overflow: hidden;
	}

	.shop-info-wrapper {
		display: flex;
		background-color: transparent;
		padding: 28rpx 24rpx 0rpx;
		margin-bottom: 10rpx;
	}
	
	.shop-info {
		flex: 1;
		display: flex;
		flex-direction: column;
	}
	
	.shop-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 12rpx;
	}
	
	.shop-name {
		line-height: 36rpx;
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		flex: 1;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}
	
	.shop-rating {
		display: flex;
		align-items: center;
	}
	
	.rating-number {
		font-size: 24rpx;
		color: #FF5A5F;
		margin-right: 10rpx;
	}

	.shop-image {
		width: 130rpx;
		height: 130rpx;
		border-radius: 8rpx;
		margin-right: 20rpx;
	}

	.shop-meta {
		display: flex;
		margin-bottom: 15rpx;
	}

	.delivery-info {
		line-height: 24rpx;
		font-size: 24rpx;
		color: #666;
		margin-right: 20rpx;
	}

	.delivery-fee {
		line-height: 24rpx;
		font-size: 24rpx;
		color: #666;
	}

	.shop-tags {
		display: flex;
		flex-wrap: wrap;
		margin-bottom: 10rpx;
	}

	.tag {
		font-size: 20rpx;
		color: #f7931e;
		padding: 4rpx 12rpx;
		border: 1rpx solid #f7931e;
		border-radius: 20rpx;
		margin-right: 10rpx;
	}

	.signature-foods-wrapper {
		display: block;
		/* 让子元素不换行 */
		white-space: nowrap; 
		vertical-align: top;
		
		padding: 0 0 24rpx 24rpx;
		width: 100%;
	}

	.food-card {
		/* 子元素横向排列 */
		display: inline-block; 
		width: 160rpx;
		margin-right: 15rpx;
	}

	.food-image-wrapper {}

	.food-image {
		width: 160rpx;
		height: 160rpx;
		border-radius: 8rpx;
	}

	.food-name {
		height: 32rpx;
		font-size: 26rpx;
		line-height: 26rpx;
	}
	.ellipsis {
		/* 禁止换行 */
		white-space: nowrap;
		/* 超出部分隐藏 */
		overflow: hidden;
		/* 显示省略号 */
		text-overflow: ellipsis;
	}
	
	.food-price {
		height: 32rpx;
		font-size: 26rpx;
		line-height: 26rpx;
		font-weight: bold;
		color: #FF5A5F;
	}

	.enter-shop-button {
		/* background-color: #000; */
		display: inline-block;
		height: 224rpx;
		width: 24rpx;
		padding: 0 20rpx;
		margin-right: 40rpx;

		writing-mode: vertical-rl;
		color: #ccc;
		font-size: 24rpx;
		line-height: 24rpx;
		text-align: center;
	}
</style>