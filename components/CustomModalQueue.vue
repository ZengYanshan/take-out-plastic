<template>
	<view>
		<view v-for="modal in modals" :key="modal.id" class="custom-modal">
			<view class="modal-mask" @click="handleCancel(modal.id)" />

			<view class="modal-container">
				<view class="modal-header">
					<text class="title">{{ modal.title }}</text>
				</view>

				<view class="modal-content">
					<rich-text :nodes="modal.content" />
				</view>

				<view class="modal-footer">
					<button class="btn cancel-btn" @click="handleCancel(modal.id)">
						{{ modal.cancelText || '取消' }}
					</button>
					<button class="btn confirm-btn" @click="handleConfirm(modal.id)">
						{{ modal.confirmText || '确认' }}
					</button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	// 弹窗队列管理器
	const modalQueue = [];
	let isProcessing = false;

	export default {
		name: 'CustomModalQueue',
		data() {
			return {
				modals: []
			};
		},
		methods: {
			// 显示弹窗并返回Promise
			showModal(config) {
				return new Promise((resolve, reject) => {
					const modal = {
						...config,
						id: Date.now() + Math.random(),
						resolve,
						reject
					};
					this.modals.push(modal);
				});
			},

			// 关闭弹窗
			closeModal(modalId, action, result) {
				const index = this.modals.findIndex(m => m.id === modalId);
				if (index === -1) return;

				const modal = this.modals[index];

				// 根据action调用对应的回调函数
				if (action === 'confirm' && modal.onConfirm) {
					modal.onConfirm(result);
				} else if (action === 'cancel' && modal.onCancel) {
					modal.onCancel(result);
				}

				// 解析Promise
				if (action === 'confirm') {
					modal.resolve(result);
				} else {
					modal.reject(new Error('用户取消'));
				}

				// 从队列中移除
				this.modals.splice(index, 1);
			},

			// 处理确认操作
			handleConfirm(modalId) {
				this.closeModal(modalId, 'confirm', true);
			},

			// 处理取消操作
			handleCancel(modalId) {
				this.closeModal(modalId, 'cancel', false);
			}
		}
	}
</script>

<style scoped>
	.custom-modal {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		z-index: 1000;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.modal-mask {
		position: absolute;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0.5);
		z-index: 1;
	}

	.modal-container {
		width: 80%;
		max-height: 80vh;
		background: white;
		border-radius: 10rpx;
		overflow: hidden;
		z-index: 2;
		display: flex;
		flex-direction: column;
	}

	.modal-header {
		margin-top: 40rpx;
		padding: 10rpx 40rpx;
		/* border-bottom: 1rpx solid #eee; */
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.title {
		font-size: 36rpx;
		font-weight: bold;
		text-align: center;
		flex: 1;
		color: #f7931e;
	}

	.modal-content {
		margin-bottom: 20rpx;
		padding: 20rpx 40rpx;
		max-height: 50vh;
		overflow-y: auto;
		font-size: 32rpx;
		line-height: 1.6;
		color: #666;
	}

	.modal-footer {
		display: flex;
		border-top: 1rpx solid #eee;
	}

	.btn {
		flex: 1;
		height: 90rpx;
		line-height: 90rpx;
		font-size: 28rpx;
		border-radius: 0;
	}

	.btn::after {
		border: none;
	}

	.cancel-btn {
		background-color: transparent;
		color: #ccc;
	}

	.confirm-btn {
		background-color: #f7931e;
		color: #fff
	}
</style>