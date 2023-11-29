# dragTemp
uniapp 微信小程序 拖拽组件、可拖拽 插入图片或插入文案拖拽

插件只是一个拖拽外壳，可在此基础上根据业务拓展添加内容

### 文档：

##### 		props 属性：

​			id 唯一标识 type: String

​			z-index 拖拽壳层级权重 type: Number

​			left 初始化定位X坐标 type: Number

​			top 初始化定位Y坐标 type: Number

​			width 拖拽容器默认宽度 type: Number

​			height 拖拽容器默认高度 type: Number

​			limit-min-width 限制缩放最小宽度 type: Number

​			limit-min-height 限制缩放最小高度 type: Number

​			limit-max-width 限制缩放最大宽度 type: Number

​			limit-max-height 限制缩放最大高度 type: Number

​			limit-x 限制可移动的X范围 type: Array

​			limit-y 限制可移动的Y范围 type: Array

​			interval-range-limit 限制多组可移动范围 type: Array	[[x0,y0],[x1,y1]]

​			scroll-top 当前滚动条高度 用于计算在出现滚动条的时候滑动到底部可添加插件到可视区范围内 type: Number

##### 		slot 插槽: 

​			content 拖拽内容区

##### 		event事件：

​			click 点击退拽内容区触发

​			delete 点击删除按钮触发

			change 位置/大小发生变化触发

##### 		实例获取

 1. 通过绑定ref 获取组件实例

    `this.$refs[id].getAttribute()`

 2. 通过id 获取组件实例 （小程序）

    `this.selectComponent("#id").$vm.getAttribute()`

    getAttribute

    返回当前组件的一些基础信息

​			{
​					id,
​					width,
​					height,
​					left,
​					top,
​					fontSize
​			}

#### **使用案例**

以下案例并非完整代码 请自行参考使用

###### 插入文字（文字不可缩放）

```vue
<dragTemp :id="id" :width="100" :height="30" :limitX="limitX" :limitY="limitY" :limitX="limitX" :limitY="limitY" :limit-min-width="50" :limit-min-height="20" :limit-max-height="60" :limit-max-width="200" :scrollTop="scrollTop"
		@click="showDate = true" @delete="(id) => { $emit('delete',id) }">
        <template v-slot:content>
            <view class="text-no-wrap ps-rl" style="width: 100%;height: 100%;">
                <text v-if="selectDate" class="text-ps-ct">{{ selectDate }}</text>
                <view v-else class="textbtn">
                    <view class="fl-ai-jc-ct text-no-wrap text-ps-ct">
                        <image src="@/static/icons/line-date.png" style="width: 36rpx;height: 36rpx;" mode=""></image>
                        <text>签署日期</text>
                    </view>
                </view>
            </view>
        </template>
</dragTemp>
```

###### 插入图片形式

```vue
<dragTemp :id="id" :width="100" :height="30" :limitX="limitX" :limitY="limitY" :limitX="limitX" :limitY="limitY" :limit-min-width="50" :limit-min-height="20" :limit-max-height="60" :limit-max-width="200" :scrollTop="scrollTop"
		@click="show = true" @delete="(id) => { $emit('delete',id) }">
		<template v-slot:content>
			<view style="width: 100%;height: 100%;">
				<previewImage v-if="signPath" :src="signPath" width="100%" height="100%" mode="aspectFit" />
				<view v-else class="textbtn" >
					<view class="text fl-jc-ct fl-ai-ct">
						<image src="@/static/icons/pansou.png" class="mr-10" mode=""></image>
						<text>电子签名</text>
					</view>
				</view>
			</view>
		</template>
</dragTemp>
```

###### interval-range-limi用途说明
	举例：多页签署合同，每页底部需要一个页眉提示栏，页眉提示栏是不可拖拽组件到上面的，所以需要限制该范围的拖拽，触发条件在touchend拖拽结束后执行
	以下代码并不完全，不能直接运行，仅供参考
```vue
<!-- 父组件调用 -->
<view id="pagerChunk" class="pager">
	<view
		v-for="(item,index) in signImageList" 
		:key="'docPager' + index" 
		class="pager-item">
		<image
			:id="'docPager' + index" 
			:src="getPic(item)" 
			mode="" 
			class="page-doc-image"
			:style="docPagerStyle['docPager' + index]" 
			@load="(e) => { imageLoad(e, 'docPager' + index) }"></image>
			<view class="canvas-list">
				<canvas :canvas-id="'docCanvas_' + 'docPager' + index" :style="canvasStyle"></canvas>
			</view>
		<view class="pager-num font-size-24 font-tips-color">
			{{ index + 1 }} / {{ signImageList.length }}
		</view>
	</view>
	<addSingItem v-for="item in signature" :id="item" :ref="item" :key="item" :scrollTop="scrollTop"
		:limitX="[0, limitWidthHeight.width]" :limitY="[0, limitWidthHeight.height]" :intervalRangeLimi="intervalRangeLimi" @delete="deleteItem">
	</addSingItem>
	<addDateItem v-for="item in dateList" :id="item" :ref="item" :key="item" :scrollTop="scrollTop"
		:limitX="[0, limitWidthHeight.width]" :limitY="[0, limitWidthHeight.height]" :intervalRangeLimi="intervalRangeLimi" @delete="deleteDate">
	</addDateItem>
</view>
```

```vue
<!-- 根据业务封装的签名子组件 -->
<az-drag-temp :id="id" :width="100" :height="30" :limitX="limitX" :limitY="limitY" :limitX="limitX" :limitY="limitY" :limit-min-width="50" :limit-min-height="20" :limit-max-height="60" :limit-max-width="200" :intervalRangeLimi="intervalRangeLimi" :scrollTop="scrollTop"
	@click="show = true" @delete="(id) => { $emit('delete',id) }">
	<template v-slot:content>
		<view style="width: 100%;height: 100%;">
			<previewImage v-if="signPath" :src="signPath" width="100%" height="100%" mode="aspectFit" />
			<view v-else class="textbtn" >
				<view class="text fl-jc-ct fl-ai-ct">
					<image src="@/static/icons/pansou.png" class="mr-10" mode=""></image>
					<text>电子签名</text>
				</view>
			</view>
		</view>
	</template>
</az-drag-temp>
```

```vue
<!-- 根据业务封装的日期选择组件 -->
<az-drag-temp :id="id" :width="100" :height="30" :limitX="limitX" :limitY="limitY" :limitX="limitX" :limitY="limitY" :limit-min-width="50" :limit-min-height="20" :limit-max-height="60" :limit-max-width="200" :intervalRangeLimi="intervalRangeLimi" :scrollTop="scrollTop"
	@click="showDate = true" @delete="(id) => { $emit('delete',id) }">
	<template v-slot:content>
		<view class="text-no-wrap ps-rl" style="width: 100%;height: 100%;">
			<text v-if="selectDate" style="font-family: '仿宋体=FangSong,serif';" class="text-ps-ct">{{ selectDate }}</text>
			<view v-else class="textbtn">
				<view class="fl-ai-jc-ct text-no-wrap text-ps-ct">
					<image src="@/static/icons/line-date.png" style="width: 36rpx;height: 36rpx;" mode=""></image>
					<text>签署日期</text>
				</view>
			</view>
		</view>
	</template>
</az-drag-temp>
```

```javascript
computed: {
	// 拖拽每页页眉不可出现的范围
	intervalRangeLimi() {
		// previewImageList 是每页的图片url列表
		// docPagerStyle 是每页的样式 用于获取宽高
		let arr = []
		let range = 24 // 页眉的高度 最好是固定死的
		for(let index in this.previewImageList) {
			let item = this.docPagerStyle['docPager' + index]
			let width = item.width.replace('px', '') * 1
			let height = item.height.replace('px', '') * 1
			let posSY = height * (index + 1) 
			let posEY = height * (index + 1) + range * (index + 1)
			arr.push([
				[0, posSY],
				[width, posEY]
			])
		}
		return arr
	},
	...
}
...
methods: {
	/**
	 * 文档图片加载完毕
	
	 * @param {Object} res
	 */
	imageLoad(res, id) {
		// 纸张大小 原图大小
		this.docWidth = res.detail.width
		this.docHeight = res.detail.height
		// 设置实际在客户端展示的纸张大小
		this.$set(this.docPagerStyle, id, {
			width: this.screenWidth + 'px',
			height: res.detail.height / res.detail.width * this.screenWidth + 'px'
		})
		this.$nextTick(() => {
			// 获取最终文档所有图片在页面展示的大小
			const query = uni.createSelectorQuery().in(this)
			query.select('#pagerChunk').boundingClientRect(data => {
				// 签署容器总宽高 用于限制拖拽壳的最大最小拖拽范围
				this.limitWidthHeight.width = data.width
				this.limitWidthHeight.height = data.height
			}).exec()
		})
	},
	...
}
```
