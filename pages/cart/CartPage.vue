<template>
	<!-- 自定义导航栏 
	<view class="status-bar" style="height: var(--status-bar-height); width: 100%;" />
	<view class="custom-navbar">
		<image src="/static/images/icon-back.png" class="back-icon" @click="goBack" />
		<text class="page-title">购物车</text>
	</view>-->

	<view class="cart-page">
		<!-- 空购物车状态 -->
		<view v-if="groupedCart.length === 0" class="empty-cart">
			<image src="/static/images/empty-cart.png" class="empty-icon" />
			<text class="empty-text">购物车还是空的，快去挑选美食吧！</text>
			<button class="go-shopping-btn" @click="goToHome">去逛逛</button>
		</view>

		<!-- 有商品状态 -->
		<scroll-view v-else scroll-y show-scrollbar="false" class="cart-scroll">
			<!-- 商店分组 -->
			<view v-for="(shopGroup, index) in groupedCart" :key="index" class="shop-group">
				<!-- 商店头部 -->
				<view class="shop-header" @click="goToShopDetail(shopGroup.shop.id)">
					<image :src="shopGroup.shop.image" class="shop-image" />
					<text class="shop-name">{{ shopGroup.shop.name }}</text>
					<text class="delivery-fee">外包装降解另需{{ shopGroup.shop.deliveryFee }}年</text>
				</view>

				<!-- 食物列表 -->
				<view class="food-list">
					<view v-for="food in shopGroup.foods" :key="food.id" class="food-item"
						@click="goToFoodDetail(shopGroup.shop.id, food.id)">
						<image :src="food.image" class="food-image" mode="aspectFill" />
						<view class="food-info">
							<text class="food-name">{{ food.name }}</text>
							<text class="food-price">{{ food.price*food.count }}年</text>
							<!-- <text class="food-desc">{{ food.description }}</text> -->
						</view>
						<view class="food-count">
							<food-count-controller :food="food" @add='onAdd' @sub='onSub' />
						</view>
						<!-- <view class="delete-btn" @click="removeFood(food)">删除</view> -->
					</view>
				</view>
			</view>
		</scroll-view>

		<!-- 底部结算栏 -->
		<view v-if="groupedCart.length > 0" class="bottom-bar content">
			<view class="content-left">
				<!-- 总价 -->
				<view class="total-price-wrapper">
					<text class="total-price" :class="{'highlight': totalPrice > 0}">
						{{ totalPrice }}年
					</text>
					<!-- 配送费 -->
					<text class="total-delivery-fee">包装降解另需{{ totalDeliveryFee }}年</text>
				</view>
				<!-- <text class="total-price-row">总降解时间: <text class="total-price">{{ totalPrice }}年</text></text> -->
			</view>
			<view class="content-right">
				<view class="pay-btn" @click="gotoPay(totalPrice)">去支付</view>
			</view>
		</view>
	</view>

	<!-- 支付弹窗 -->
	<custom-modal-queue ref="modalQueue" />
</template>

<script>
	import SHOPS from '@/static/data/shops.js';
	import FOODS from '@/static/data/foods.js';
	import FoodCountController from '@/components/FoodCountController.vue';
	import CustomModalQueue from '@/components/CustomModalQueue.vue';

	export default {
		components: {
			FoodCountController,
			CustomModalQueue
		},
		computed: {
			groupedCart() {
				console.log("cartData = ", this.$store.state.cart.cartData);
				const shopMap = new Map();
				this.$store.state.cart.cartData.forEach(item => {
					const food = FOODS.find(f => f.id === item.foodId);
					if (!food) return;

					const shop = SHOPS.find(s =>
						s.menu.some(cat =>
							cat.items.some(i => i.foodId === item.foodId)
						)
					);


					if (shop) {
						if (!shopMap.has(shop.id)) {
							shopMap.set(shop.id, {
								shop,
								foods: []
							});
						}

						const shopGroup = shopMap.get(shop.id); // 引用类型，可修改shopMap
						shopGroup.foods.push({
							...food,
							count: item.count
						});
					}
				});

				let _groupedCart = Array.from(shopMap.values());
				console.log("groupedCart = ", _groupedCart);
				return _groupedCart;
			},
			totalPrice() {
				return this.groupedCart.reduce(
					(total, shopGroup) => total +
					shopGroup.foods.reduce(
						(sum, food) => sum + food.price * food.count,
						0
					),
					0
				);
			},
			totalDeliveryFee() {
				return this.groupedCart.reduce(
					(total, shopGroup) => total + shopGroup.shop.deliveryFee,
					0
				);
			}
		},
		methods: {
			goToShopDetail(shopId) {
				let _url = `/pages/shop/ShopDetailPage?shopId=${shopId}`;
				uni.navigateTo({
					url: _url
				});
			},
			goToFoodDetail(shopId, foodId) {
				let _url = `/pages/shop/ShopDetailPage?shopId=${shopId}&foodId=${foodId}`;
				uni.navigateTo({
					url: _url
				});
			},
			async onAdd(food) {
				await this.$store.dispatch('cart/addFood', {
					foodId: food.id
				});
			},
			async onSub(food) {
				await this.$store.dispatch('cart/subFood', {
					foodId: food.id
				});
			},
			// 删除食物
			removeFood(food) {
				this.subFood({
					foodId: food.id,
					count: food.count
				});
			},

			// 跳转首页
			goToHome() {
				uni.switchTab({
					url: '/pages/home/HomePage'
				});
			},

			// 去支付
			async gotoPay(totalPrice) {
				try {
					// 第一个弹窗
					const firstResult = await this.$refs.modalQueue.showModal({
						title: '支付提示',
						content: `支付降解时间：${totalPrice}年`,
						confirmText: '下一步',
						cancelText: '取消支付',
						confirmColor: '#f7931e'
					});

					// 第二个弹窗（只有第一个弹窗确认后才会执行）
					const secondResult = await this.$refs.modalQueue.showModal({
						title: '贷款确认',
						content: `您的余额不足！<br>推荐您使用贷款：<br>是否向地球贷款${totalPrice}年？`,
						confirmText: '立即贷款',
						cancelText: '放弃支付',
						confirmColor: '#f7931e'
					});

					// 两个弹窗都确认后的逻辑
					console.log('用户确认贷款并支付');
					this.onConfirm();

				} catch (error) {
					// 用户取消支付
				}
			},
			onConfirm() {
				uni.showModal({
					title: '系统提示',
					content: '贷款并不会真的发生，因为这不是现实世界的外卖APP。',
					confirmColor: "#f7931e"
				});
			},
		}
	};
</script>

<style scoped>
	/* 自定义导航栏 
	.custom-navbar {
		display: flex;
		align-items: center;
		padding: 15rpx 30rpx;
		background-color: #FF5A5F;
		color: white;
	}
	.back-icon {
		width: 40rpx;
		height: 40rpx;
		margin-right: 20rpx;
	}
	.page-title {
		flex: 1;
		font-size: 36rpx;
		font-weight: bold;
	}*/

	.cart-page {
		display: flex;
		flex-direction: column;
		background-color: #f5f5f5;

		padding: 24rpx 20rpx;
	}

	.empty-cart {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100%;
		text-align: center;
	}

	.empty-icon {
		width: 200rpx;
		height: 200rpx;
		margin-bottom: 40rpx;
	}

	.empty-text {
		font-size: 32rpx;
		color: #999;
		margin-bottom: 40rpx;
	}

	.go-shopping-btn {
		background-color: #f7931e;
		color: white;
		font-size: 32rpx;
		line-height: 32rpx;
		font-weight: bold;
		padding: 32rpx 40rpx;
		border-radius: 8rpx;
	}

	.cart-scroll {
		flex: 1;
		overflow: auto;
	}

	.shop-group {
		background-color: white;
		border-radius: 16rpx;
		overflow: hidden;
		margin-bottom: 24rpx;
	}

	.shop-header {
		display: flex;
		align-items: center;
		padding: 20rpx;
		border-bottom: 1rpx solid #eee;
	}

	.shop-image {
		width: 80rpx;
		height: 80rpx;
		border-radius: 8rpx;
		margin-right: 20rpx;
	}

	.shop-name {
		font-size: 32rpx;
		font-weight: bold;
		margin-right: 20rpx;
	}

	.delivery-fee {
		font-size: 28rpx;
		color: #999;
	}

	.food-list {
		padding: 0 20rpx;
	}

	.food-item {
		display: flex;
		align-items: center;
		padding: 20rpx 0;
		border-bottom: 1rpx solid #eee;
	}

	.food-image {
		width: 120rpx;
		height: 120rpx;
		border-radius: 8rpx;
		margin-right: 20rpx;
	}

	.food-info {
		flex: 1;

		height: 120rpx;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
	}

	.food-name {
		line-height: 32rpx;
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 10rpx;
	}

	.food-price {
		font-size: 32rpx;
		font-weight: bold;
		color: #FF5A5F;
	}

	.food-desc {
		font-size: 24rpx;
		color: #999;
		margin-top: 10rpx;
	}

	.food-count {
		display: flex;
		align-items: center;
		margin-right: 5rpx;
	}

	.delete-btn {
		font-size: 28rpx;
		color: #999;
	}

	.content {
		display: flex;
	}

	.content-left {
		flex: 1;
		display: flex;
		align-items: center;
	}

	.content-right {
		flex: 0 0 210rpx;
		width: 210rpx;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.bottom-bar {
		position: fixed;
		/* 关键修改：使用CSS变量动态适配tabBar高度 */
		bottom: var(--window-bottom, 0);
		left: 0;
		right: 0;
		height: 100rpx;
		/* margin: 0 10rpx; */
		/* padding: 0 30rpx; */
		background-color: #fff;
		box-shadow: 0 -2rpx 6rpx rgba(0, 0, 0, 0.1);
		/* 确保层级高于tabBar */
		z-index: 999;
	}

	.total-price-wrapper {
		flex: 1;
		height: 100%;
		
		display: flex;
		flex-direction: column;
		padding: 0 20rpx;
	}

	.total-price {
		color: #FF5A5F;
		display: inline-block;
		vertical-align: top;
		margin: 14rpx 0 8rpx;
		font-weight: bold;
		font-size: 40rpx;
		line-height: 40rpx;
	}
	
	.total-delivery-fee {
		display: inline-block;
		vertical-align: top;
		line-height: 24rpx;
		font-size: 24rpx;
		color: #999;
	}

	.pay-btn {
		width: 100%;
		height: 100%;
		line-height: 100rpx;
		text-align: center;
		font-weight: 700;
		font-size: 28rpx;
		background: #f7931e;
		color: #fff;
	}
</style>