<template>
	<view :ref="id" :id="id" class="drag-temp" :style="ballStyle">
		<view class="mask" @touchstart.stop="touchstart" @touchmove.stop="(event) => {throttle(event, touchmove)}" @touchend.stop="touchend"></view>
		<slot name="content"></slot>
		<view v-if="!$slots.content" class="textbtn">
			<text class="text">点击添加内容</text>
		</view>
		<view v-if="canDelete" class="delete" :class="{'left-position': leftPosition}" @click.stop="deleteItem">
			<view class="icon"></view>
		</view>
		<view v-if="canScale" class="scale" :class="{'left-position': leftPosition}" @touchstart.stop="(e) => { touchstart(e, true) }" @touchmove.stop="(event) => {throttle(event, touchmove)}" @touchend.stop="touchend">
			<view class="icon"></view>
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
			// 拖拽壳层级权重
			zIndex: {
				type: Number,
				default: 10
			},
			// 是否可拖拽
			canDrag: {
				type: Boolean,
				default: true
			},
			// 是否可缩放
			canScale: {
				type: Boolean,
				default: true
			},
			// 是否可删除
			canDelete: {
				type: Boolean,
				default: true
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
			limitMinWidth: {
				type: Number,
				default: 0
			},
			// 限制缩放最小高度
			limitMinHeight: {
				type: Number,
				default: 0
			},
			// 限制缩放最大宽度
			limitMaxWidth: {
				type: Number,
				default: 0
			},
			// 限制缩放最大高度
			limitMaxHeight: {
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
			// 限制多组可移动范围
			intervalRangeLimit: {
				type: Array,
				default: () => {
					return []
				}
			},
			// 当前滚动条高度
			scrollTop: {
				type: Number,
				default: 0
			},
			// 节流间隔时长
			throttleDelay: {
				type: Number,
				default: 20
			}
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
				isMove: false,
				scale: false,
				timer: null
			}
		},
		mounted() {
			if (this.width) this.ballStyle.width = this.width + 'px'
			if (this.height) this.ballStyle.height = this.height + 'px'
			if (this.zIndex) this.ballStyle.zIndex = this.zIndex
			if (this.left) this.ballStyle.left = this.left + 'px'
			if (this.top) this.ballStyle.top = this.top + 'px'
			if (this.scrollTop !== 0) this.ballStyle.top = this.ballStyle.top.replace('px', '') * 1 + this.scrollTop + 'px'
		},
		methods: {
			// 节流处理
			throttle(event, fn) {
				clearTimeout(this.timer)
				this.timer = setTimeout(function () {
					fn(event);
					this.timer = null;
				}, this.throttleDelay);
			},
			selectSign(item) {
				this.signPath = item.filePath
				this.show = false
			},
			touchstart(event, scale = false) {
				this.scale = scale
				// 阻止事件冒泡
				// event.stopPropagation();
				this.startX = event.touches[0].clientX
				this.startY = event.touches[0].clientY
			},
			touchmove(event) {
				this.endX = event.touches[0].clientX
				this.endY = event.touches[0].clientY
				if (this.endX - this.startX != 0 || this.endY - this.startY != 0) {
					// X 或 Y 定位有变动则证明正在移动
					this.isMove = true
				}
				if (this.scale) {
					// 缩放
					if (this.ballStyle.width.replace('px', '') * 1 + this.endX - this.startX <= this.limitMinWidth) {
						this.ballStyle.width = this.limitMinWidth + 'px'
					} else if (this.limitMaxWidth && this.ballStyle.width.replace('px', '') * 1 + this.endX - this.startX >= this.limitMaxWidth) {
						this.ballStyle.width = this.limitMaxWidth + 'px'
					} else {
						this.ballStyle.width = this.ballStyle.width.replace('px', '') * 1 + this.endX - this.startX + 'px'
					}
					if (this.ballStyle.height.replace('px', '') * 1 + this.endY - this.startY <= this.limitMinHeight) {
						this.ballStyle.height = this.limitMinHeight + 'px'
					} else if (this.limitMaxHeight && this.ballStyle.height.replace('px', '') * 1 + this.endY - this.startY >= this.limitMaxHeight) {
						this.ballStyle.height = this.limitMaxHeight + 'px'
					} else {
						this.ballStyle.height = this.ballStyle.height.replace('px', '') * 1 + this.endY - this.startY + 'px'
					}
				} else {
					if (!this.canDrag) return
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
					this.judgeRange()
					this.$emit('change', this.getAttribute())
					return
				}
				this.$emit('click', this.id)
			},
			/**
			 * 判断拖拽容器位置是否在禁拖区 多组禁拖区
			 */
			judgeRange() {
				if (this.intervalRangeLimit.length) {
					let wrapBox = [
						[
							this.ballStyle.left.replace('px', '') * 1, 
							this.ballStyle.top.replace('px', '') * 1
						],
						[
							this.ballStyle.left.replace('px', '') * 1 + this.ballStyle.width.replace('px', '') * 1, 
							this.ballStyle.top.replace('px', '') * 1 + this.ballStyle.height.replace('px', '') * 1
						]
					]
					for(let item of this.intervalRangeLimit) {
						let wrapBoxWidth = wrapBox[1][0] - wrapBox[0][0]
						let wrapBoxHeight = wrapBox[1][1] - wrapBox[0][1]
						// x轴限制范围处理
						if (
							(wrapBox[0][0] >= item[0][0] && wrapBox[0][0] <= item[1][0]) ||
							(wrapBox[1][0] >= item[0][0] && wrapBox[1][0] <= item[1][0]) ||
							(item[0][0] >= wrapBox[0][0] && item[0][0] <= wrapBox[1][0]) ||
							(item[1][0] >= wrapBox[0][0] && item[1][0] <= wrapBox[1][0]) 
						) {
							this.newDragPositionX(item, wrapBox)
						}
						// y轴限制范围处理
						if (
							(wrapBox[0][1] >= item[0][1] && wrapBox[0][1] <= item[1][1]) ||
							(wrapBox[1][1] >= item[0][1] && wrapBox[1][1] <= item[1][1]) ||
							(item[0][1] >= wrapBox[0][1] && item[0][1] <= wrapBox[1][1]) ||
							(item[1][1] >= wrapBox[0][1] && item[1][1] <= wrapBox[1][1]) 
						) {
							this.newDragPositionY(item, wrapBox)
						}
					}
				}
			},
			newDragPositionX(item, wrapBox) {
				let tryNewX = item[0][0] - (wrapBox[1][0] - wrapBox[0][0])
				if (tryNewX >= this.limitX[0] && tryNewX <= this.limitX[1]) {
					// 新赋值的x范围在可移动范围可直接赋值
					this.ballStyle.left = tryNewX + 'px'
				} else {
					tryNewX = item[1][0] + (wrapBox[1][0] - wrapBox[0][0])
					if (tryNewX >= this.limitX[0] && tryNewX <= this.limitX[1]) {
						// 新赋值的x范围在可移动范围可直接赋值
						this.ballStyle.left = tryNewX + 'px'
					} else {
						// x轴两边都不可容纳 保持原状 移动Y轴来控制新的可拖拽位置
					}
				}
			},
			newDragPositionY(item, wrapBox) {
				let tryNewY = item[0][1] - (wrapBox[1][1] - wrapBox[0][1])
				if (tryNewY >= this.limitY[0] && tryNewY <= this.limitY[1]) {
					// 新赋值的y范围在可移动范围可直接赋值
					this.ballStyle.top = tryNewY + 'px'
				} else {
					tryNewY = item[1][1] + (wrapBox[1][1] - wrapBox[0][1])
					if (tryNewY >= this.limitY[0] && tryNewY <= this.limitY[1]) {
						// 新赋值的y范围在可移动范围可直接赋值
						this.ballStyle.top = tryNewY + 'px'
					} else {
						// y轴两边都不可容纳 保持原状 移动X轴来控制新的可拖拽位置
					}
				}
			},
			/**
			 * 删除操作
			 */
			deleteItem() {
				this.$emit('delete', this.id)
			},
			/**
			 * 获取拖拽容器属性
			 */
			getAttribute() {
				return {
					id: this.id,
					width: this.ballStyle.width,
					height: this.ballStyle.height,
					left: this.ballStyle.left,
					top: this.ballStyle.top
				}
			},
		}
	}
</script>

<style lang="scss" scoped>
	@mixin icon-position($top,$bottom,$transformR,$transformL,$image) {
		box-sizing: border-box;
		position: absolute;
		right: 30rpx;
		top: $top;
		bottom: $bottom;
		width: 96rpx;
		height: 96rpx;
		padding: 30rpx;
		transform: $transformR;
		.icon{
			width: 100%;
			height: 100%;
			padding: 5rpx;
			border-radius: 10rpx;
			background-color: $uni-color-primary;
			&::after{
				content: '';
				display: block;
				width: 100%;
				height: 100%;
				background-image: url($image);
				background-size: 100% 100%;
			}
		}
		&.left-position {
			left: 30rpx;
			right: auto;
			transform: $transformL;
		}
	}
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
			@include icon-position(30rpx, auto, translate(100%, -100%), translate(-100%, -100%), "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAAXNSR0IArs4c6QAAAQlJREFUWEftlksOwzAIROFkbU+W5mRpT0aFZFtRGmCwW2XjbGOYZ75muvjji/VpAswIuBEQkTsR3Zh5zRZrtSWiNzO/LPsIQHaGTxSkiG/VlplNnQyA+gshjuIF4mFFIQvgQhjiNBIBrYEWSi8dljgRmbdXf2EbOo5bOnrFIQA95EFolRtRcm/eChRtLwfizAUkDkegKoAQsHgaIEiH/k6J/wMgnBPHfIVdsDcAU5CCgAFA8coLQ0AAXp8XRWhYnbVLCIAMGWRY9W5DaxR/VXsvRM8yMlvtp8sou1aDYWUWJRoBfdGs3svGa9fudVwm34K+hI6FJiLL0JMMXVQj58I2HHGO2E6AGYEPjSyeIYlc/BUAAAAASUVORK5CYII=")
		}

		.scale {
			@include icon-position(auto, 30rpx, translate(100%, 100%), translate(-100%, 100%), "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAAAXNSR0IArs4c6QAAAc1JREFUWEfd179KHFEUx/HvIUUKwSKQgKJFiEWwUFCJREzQgAERixTmEWxFX0B9AdMJSZ3KP8+QbQJJkSIiqZI38AFSCD85chZ2x5nZcffeLbzt3HvuZ+6fc2ZMUgtYBh7RvH0ys73m3at7mqT/wOM+gs2a2UUf47qGOED9BDEz62dccUwVwLflsGaCSzO7ygnw2Kdm9jHFJHUxiivwE1jsGJAdUQRsADvA+2EhioBV4DdwBrwbBuIOwMxakp4G4m1uRCnAJ5U0FoilnIhKQCAmA/EqF6IWEIjngZjLgegJCMQUcA7MdCBW/bwMmicccATsAr/MbKEqoKSXwFdgHmiZmd+YgdttPpc007SwSFpJ8eZteZKCMsgyPCxA0+2R5N8fE2b2L9kKSPoGrPSqopK82B0Dfq2/JAH4mwMOaLfKKirpANhPfgglnQBbvRA5Af5Rewp8qENkA0Q+8cPliM0qRFZAIEYCsV6GyA4IxGgUsLUiYiiAQDwJRGfN8O35k+UWlKVjSc8C8abjuQOmk1/Dmio6HojXZX2SJKJ24EhIZfP4SviPjpf0rpYMIOlH4Z+iUZFMApD0AvjbaMbuTtdJAHHqPwPb90BcA99vAL532kBdSSq5AAAAAElFTkSuQmCC")
		}
	}
</style>