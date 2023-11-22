# dragTemp
uniapp 微信小程序 拖拽组件、可拖拽 插入图片或插入文案拖拽

插件只是一个拖拽外壳，可在此基础上根据业务拓展添加内容

### 文档：

##### 		props 属性：

​			id 唯一标识 type: String

​			left 初始化定位X坐标 type: Number

​			top 初始化定位Y坐标 type: Number

​			width 拖拽容器默认宽度 type: Number

​			height 拖拽容器默认高度 type: Number

​			limit-width 限制缩放最小宽度 type: Number

​			limit-height 限制缩放最小高度 type: Number

​			limit-x 限制可移动的X范围 type: Array

​			limit-y 限制可移动的Y范围 type: Array

​			scroll-top 当前滚动条高度 用于计算在出现滚动条的时候滑动到底部可添加插件到可视区范围内 type: Number

##### 		slot 插槽: 

​			content 拖拽内容区

##### 		event事件：

​			click 点击退拽内容区触发

​			delete 点击删除按钮触发

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

`

```vue
<dragTemp :id="id" :width="100" :height="30" :limitX="limitX" :limitY="limitY" :limitWidth="100" :limitHeight="30" :scrollTop="scrollTop"
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

`

###### 插入图片形式

`

```vue
<dragTemp :id="id" :width="100" :height="30" :limitX="limitX" :limitY="limitY" :limitWidth="100" :limitHeight="30" :scrollTop="scrollTop"
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

`

