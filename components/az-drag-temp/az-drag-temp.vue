<template>
	<view :ref="id" :id="id" class="drag-temp" :style="ballStyle">
		<view class="mask" @touchstart.stop="touchstart" @touchmove.stop="touchmove" @touchend.stop="touchend"></view>
		<slot name="content"></slot>
		<view v-if="!$slots.content" class="textbtn">
			<text class="text">点击添加内容</text>
		</view>
		<view class="delete" :class="{'left-position': leftPosition}" @click.stop="deleteItem">
		</view>
		<view class="scale" :class="{'left-position': leftPosition}" @touchstart.stop="touchstart" @touchmove.stop="(e) => {
				touchmove(e, true)
			}" @touchend.stop="touchend">
		</view>
	</view>
</template>

<script>
	export default {
		name: 'AzDragTemp',
		props: {
			// 唯一标识id
			id: {
				type: String,
				default: 'temp_' + new Date().getTime()
			},
			// 初始化定位X坐标
			left: {
				type: Number,
				default: 100
			},
			// 初始化定位Y坐标
			top: {
				type: Number,
				default: 100
			},
			// 限制缩放最小宽度
			limitWidth: {
				type: Number,
				default: 0
			},
			// 限制缩放最小高度
			limitHeight: {
				type: Number,
				default: 0
			},
			// 宽度
			width: {
				type: Number,
				default: 100
			},
			// 高度
			height: {
				type: Number,
				default: 30
			},
			// 限制可移动的X范围
			limitX: {
				type: Array,
				default: () => {
					return [0, 0]
				}
			},
			// 限制可移动的Y范围
			limitY: {
				type: Array,
				default: () => {
					return [0, 0]
				}
			},
			// 当前滚动条高度
			scrollTop: {
				type: Number,
				default: 0
			},
		},
		data() {
			return {
				show: false,
				leftPosition: false,
				signPath: '',
				signList: [],
				ballStyle: {
					position: 'absolute',
					zIndex: '10',
					width: '100px',
					fontSize: '12px',
					height: '30px',
					lineHeight: '28px',
					textAlign: 'center',
					top: '100px',
					left: '100px'
				},
				startX: null,
				startY: null,
				endX: null,
				endY: null,
				isMove: false
			}
		},
		mounted() {
			if (this.scrollTop !== 0) {
				this.ballStyle.top = this.ballStyle.top.replace('px', '') * 1 + this.scrollTop + 'px'
			}
			if (this.width) {
				this.ballStyle.width = this.width + 'px'
			}
			if (this.height) {
				this.ballStyle.height = this.height + 'px'
			}
			if (this.left) {
				this.ballStyle.left = this.left + 'px'
			}
			if (this.top) {
				this.ballStyle.top = this.top + 'px'
			}
		},
		methods: {
			selectSign(item) {
				this.signPath = item.filePath
				this.show = false
			},
			touchstart(event) {
				// 阻止事件冒泡
				// event.stopPropagation();
				this.startX = event.touches[0].clientX
				this.startY = event.touches[0].clientY
			},
			touchmove(event, scale = false) {
				this.endX = event.touches[0].clientX
				this.endY = event.touches[0].clientY
				if (this.endX - this.startX != 0 || this.endY - this.startY != 0) {
					// X 或 Y 定位有变动则证明正在移动
					this.isMove = true
				}
				if (scale) {
					// 缩放
					if (this.ballStyle.width.replace('px', '') * 1 + this.endX - this.startX <= this.limitWidth) {
						this.ballStyle.width = this.limitWidth + 'px'
					} else {
						this.ballStyle.width = this.ballStyle.width.replace('px', '') * 1 + this.endX - this.startX + 'px'
					}
					if (this.ballStyle.height.replace('px', '') * 1 + this.endY - this.startY <= this.limitHeight) {
						this.ballStyle.height = this.limitHeight + 'px'
					} else {
						this.ballStyle.height = this.ballStyle.height.replace('px', '') * 1 + this.endY - this.startY + 'px'
					}
				} else {
					// 移动
					// 可移动范围 X
					if (this.ballStyle.left.replace('px', '') * 1 + this.endX - this.startX <= this.limitX[0]) {
						this.ballStyle.left = '0px'
						this.leftPosition = false
					} else if (this.limitX[1] && this.ballStyle.left.replace('px', '') * 1 + this.endX - this.startX + this
						.ballStyle.width.replace('px', '') * 1 >= this.limitX[1]) {
						this.ballStyle.left = this.limitX[1] - this.ballStyle.width.replace('px', '') + 'px'
					} else {
						if (this.limitX[1] - (this.ballStyle.left.replace('px', '') * 1 + this.endX - this.startX + this
								.ballStyle.width.replace('px', '') * 1) <= 26) {
							this.leftPosition = true
						} else {
							this.leftPosition = false
						}
						this.ballStyle.left = this.ballStyle.left.replace('px', '') * 1 + this.endX - this.startX + 'px'
					}
					// 可移动范围 Y
					if (this.ballStyle.top.replace('px', '') * 1 + this.endY - this.startY <= this.limitY[0]) {
						this.ballStyle.top = '0px'
					} else if (this.limitY[1] && this.ballStyle.top.replace('px', '') * 1 + this.endY - this.startY + this
						.ballStyle.height.replace('px', '') * 1 >= this.limitY[1]) {
						this.ballStyle.top = this.limitY[1] - this.ballStyle.height.replace('px', '') + 'px'
					} else {
						this.ballStyle.top = this.ballStyle.top.replace('px', '') * 1 + this.endY - this.startY + 'px'
					}
				}
				this.startX = event.touches[0].clientX
				this.startY = event.touches[0].clientY
			},
			touchend(event) {
				if (this.isMove) {
					this.isMove = false
					return
				}
				this.$emit('click')
			},
			/**
			 * 删除操作
			 */
			deleteItem() {
				this.$emit('delete', this.id)
			},
			getAttribute() {
				return {
					id: this.id,
					width: this.ballStyle.width,
					height: this.ballStyle.height,
					left: this.ballStyle.left,
					top: this.ballStyle.top,
					fontSize: this.ballStyle.fontSize
				}
			},
		}
	}
</script>

<style lang="scss" scoped>
	.drag-temp {
		border: 1px dashed rgb(38, 117, 236);

		.textbtn {
			position: relative;
			width: 100%;
			height: 100%;
			background-color: #CFDEF6;

			.text {
				position: absolute;
				left: 50%;
				top: 50%;
				width: 100%;
				transform: translate(-50%, -50%);
			}
		}

		.mask {
			position: absolute;
			left: 0;
			top: 0;
			right: 0;
			bottom: 0;
			z-index: 10;
		}

		.delete {
			position: absolute;
			right: 0;
			top: 0;
			width: 36rpx;
			height: 36rpx;
			padding: 5rpx;
			transform: translate(100%, -100%);
			background-color: $uni-color-primary;
			border-radius: 10rpx;
			&::after{
				content: '';
				display: block;
				width: 100%;
				height: 100%;
				background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAAXNSR0IArs4c6QAAAQlJREFUWEftlksOwzAIROFkbU+W5mRpT0aFZFtRGmCwW2XjbGOYZ75muvjji/VpAswIuBEQkTsR3Zh5zRZrtSWiNzO/LPsIQHaGTxSkiG/VlplNnQyA+gshjuIF4mFFIQvgQhjiNBIBrYEWSi8dljgRmbdXf2EbOo5bOnrFIQA95EFolRtRcm/eChRtLwfizAUkDkegKoAQsHgaIEiH/k6J/wMgnBPHfIVdsDcAU5CCgAFA8coLQ0AAXp8XRWhYnbVLCIAMGWRY9W5DaxR/VXsvRM8yMlvtp8sou1aDYWUWJRoBfdGs3svGa9fudVwm34K+hI6FJiLL0JMMXVQj58I2HHGO2E6AGYEPjSyeIYlc/BUAAAAASUVORK5CYII=");
				background-size: 100% 100%;
			}
			&.left-position {
				left: 0;
				right: auto;
				transform: translate(-100%, -100%);
			}
		}

		.scale {
			position: absolute;
			right: 0;
			bottom: 0;
			width: 36rpx;
			height: 36rpx;
			padding: 5rpx;
			transform: translate(100%, 100%);
			background-color: $uni-color-primary;
			border-radius: 10rpx;
			&::after{
				content: '';
				display: block;
				width: 100%;
				height: 100%;
				background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAAXNSR0IArs4c6QAAAc1JREFUWEfd179KHFEUx/HvIUUKwSKQgKJFiEWwUFCJREzQgAERixTmEWxFX0B9AdMJSZ3KP8+QbQJJkSIiqZI38AFSCD85chZ2x5nZcffeLbzt3HvuZ+6fc2ZMUgtYBh7RvH0ys73m3at7mqT/wOM+gs2a2UUf47qGOED9BDEz62dccUwVwLflsGaCSzO7ygnw2Kdm9jHFJHUxiivwE1jsGJAdUQRsADvA+2EhioBV4DdwBrwbBuIOwMxakp4G4m1uRCnAJ5U0FoilnIhKQCAmA/EqF6IWEIjngZjLgegJCMQUcA7MdCBW/bwMmicccATsAr/MbKEqoKSXwFdgHmiZmd+YgdttPpc007SwSFpJ8eZteZKCMsgyPCxA0+2R5N8fE2b2L9kKSPoGrPSqopK82B0Dfq2/JAH4mwMOaLfKKirpANhPfgglnQBbvRA5Af5Rewp8qENkA0Q+8cPliM0qRFZAIEYCsV6GyA4IxGgUsLUiYiiAQDwJRGfN8O35k+UWlKVjSc8C8abjuQOmk1/Dmio6HojXZX2SJKJ24EhIZfP4SviPjpf0rpYMIOlH4Z+iUZFMApD0AvjbaMbuTtdJAHHqPwPb90BcA99vAL532kBdSSq5AAAAAElFTkSuQmCC");
				background-size: 100% 100%;
			}
			&.left-position {
				left: 0;
				right: auto;
				transform: translate(-100%, 100%);
			}
		}
	}
</style>