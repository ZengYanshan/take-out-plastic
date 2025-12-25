<template>
	<view class="shop-info-wrapper">
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
					{{ shop.deliveryFee === 0 ? '免外包装' : `降解需${shop.deliveryFee}年` }}
				</text>
			</view>

			<view class="shop-tags">
				<text v-for="(tag, index) in shop.tags" :key="index" class="tag">
					{{ tag }}
				</text>
			</view>

		</view>
	</view>
</template>

<script>
	export default {
		name: 'ShopHeader',
		props: {
			shop: {
				type: Object,
				default: () => ({})
			}
		},
		methods: {}
	};
</script>

<style scoped>
	.shop-info-wrapper {
		display: flex;
		width: calc(100vw - 24px);
		background-color: #fff;
		padding: 24rpx 24rpx 0rpx;
		margin-bottom: 24rpx;
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
</style>