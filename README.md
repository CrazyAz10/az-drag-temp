# dragTemp
uniapp 微信小程序 拖拽组件、可拖拽 插入图片或插入文案拖拽

插件只是一个拖拽外壳，可在此基础上根据业务拓展添加内容

### 文档：

##### 		props 属性：

​			id 唯一标识 type: String

​			z-index 拖拽壳层级权重 type: Number	10

​			left 初始化定位X坐标 type: Number	100

​			top 初始化定位Y坐标 type: Number	100

​			width 拖拽容器默认宽度 type: Number	100

​			height 拖拽容器默认高度 type: Number	30

​			limit-min-width 限制缩放最小宽度 type: Number	0

​			limit-min-height 限制缩放最小高度 type: Number	0

​			limit-max-width 限制缩放最大宽度 type: Number	0

​			limit-max-height 限制缩放最大高度 type: Number	0

​			limit-x 限制可移动的X范围 type: Array	[x0,x1]

​			limit-y 限制可移动的Y范围 type: Array	[y0,y1]

​			interval-range-limit 限制多组可移动范围 type: Array	[ [[x00,y00],[x01,y01]], [[x10,y10],[x11,y11]],... ]

​			scroll-top 当前滚动条高度 用于计算在出现滚动条的时候滑动到底部可添加插件到可视区范围内 type: Number	0

​			throttle-delay 节流间隙时长(拖拽或缩放时，间隔多久触发一次响应事件) type: Number 	10	

##### 		slot 插槽: 

​			content 拖拽内容区

##### 		event事件：

​			click 点击退拽内容区触发

​			delete 点击删除按钮触发

​			change 位置/大小发生变化触发

##### 		实例获取

 1. 通过绑定ref 获取组件实例

    `this.$refs[id].getAttribute()`

 2. 通过id 获取组件实例 （小程序）

    `this.selectComponent("#id").$vm.getAttribute()`

    getAttribute返回当前组件的一些基础信息：

    ​		`{
    ​					id,
    ​					width,
    ​					height,
    ​					left,
    ​					top,
    ​					fontSize
    ​		}`

##### **使用案例**
参考demo：[https://github.com/CrazyAz10/az-drag-temp-uni-wx](https://github.com/CrazyAz10/az-drag-temp-uni-wx)
以下案例并非完整代码 请自行参考使用

##### 插入文字（文字不可缩放）

```vue
<az-drag-temp 
	:id="id" 
	:width="100" 
	:height="30" 
	:limitX="limitX" 
	:limitY="limitY" 
	:limit-min-width="50" 
	:limit-min-height="20" 
	:limit-max-width="200" 
	:limit-max-height="60" 
	:intervalRangeLimit="intervalRangeLimit" 
	:scrollTop="scrollTop"
	@click="dragClick" @change="changeTemp" @delete="deleteTemp">
	<template v-slot:content>
		<!-- 根据个人需求调整内容 -->
		<view class="content-wrap" style="width: 100%;height: 100%;">
			<image v-if="selectImagePic" :src="selectImagePic" style="width: 100%;height: 100%;" mode="aspectFit" />
			<view v-else class="textbtn" >
				<view class="default-text">
					<image src="@/static/icons/pansou.png" class="mr-10" mode=""></image>
					<text>电子签名</text>
				</view>
			</view>
		</view>
	</template>
</az-drag-temp>
```

```javascript
	export default {
		data() {
			return {
				limitX: 0,
				limitY: 0,
				scrollTop: 0,
				intervalRangeLimit: [],
				selectImagePic: ""
			}
		},
		methods: {
			dragClick() {
				// 点击了拖拽外壳
			},
			changeTemp() {
				// 拖拽外壳有变动 大小/位置
			},
			deleteTemp(id) {
				// 拖拽外壳点击了删除操作 id 是唯一标识
			}
		}
	}
```

##### interval-range-limit用途说明

格式：[
		[
			// 一个限制范围
			[起点X坐标,起点Y坐标],
			[终点X坐标,终点Y坐标]
		], // 可配置多个限制范围
		...
	]
举例：多页签署合同，每页底部需要一个页眉提示栏，页眉提示栏是不可拖拽组件到上面的，所以需要限制该范围的拖拽，触发条件在touchend拖拽结束后执行