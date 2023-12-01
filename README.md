# dragTemp
uniapp 微信小程序端拖拽组件、可拖拽 插入图片或插入文案拖拽

插件只是一个拖拽外壳，可在此基础上根据业务拓展添加内容

### 文档：

#### **props 属性：**

| 属性名称             | 描述                                                         | 类型    | 默认值                         |
| -------------------- | :----------------------------------------------------------- | ------- | ------------------------------ |
| id                   | 唯一标识                                                     | String  | 'temp_' + new Date().getTime() |
| z-index              | 拖拽壳层级权重                                               | Number  | 10                             |
| can-drag             | 是否可拖拽移动                                               | Boolean | true                           |
| can-scale            | 是否可缩放                                                   | Boolean | true                           |
| can-delete           | 是否可删除                                                   | Boolean | true                           |
| left                 | 初始化定位X坐标                                              | Number  | 100                            |
| top                  | 初始化定位Y坐标                                              | Number  | 100                            |
| width                | 拖拽容器默认宽度                                             | Number  | 100                            |
| height               | 拖拽容器默认高度                                             | Number  | 30                             |
| limit-min-width      | 限制缩放最小宽度                                             | Number  | 0                              |
| limit-min-height     | 限制缩放最小高度                                             | Number  | 0                              |
| limit-max-width      | 限制缩放最大宽度                                             | Number  | 0                              |
| limit-max-height     | 限制缩放最大高度                                             | Number  | 0                              |
| limit-x              | 限制可移动的X范围                                            | Array   | [x0,x1]                        |
| limit-y              | 限制可移动的Y范围                                            | Array   | [y0,y1]                        |
| interval-range-limit | 限制多组可移动范围(触发条件在touchend拖拽结束后执行)<br/>格式：[<br/>		[<br/>			// 一个限制范围<br/>			[起点X坐标,起点Y坐标],<br/>			[终点X坐标,终点Y坐标]<br/>		], // 可配置多个限制范围<br/>		...<br/>]<br/>用途举例：多页签署合同，每页底部需要一个页眉提示栏，页眉提示栏是不可在上面拖拽，所以需要限制该范围的拖拽 | Array   | []                             |
| scroll-top           | 当前滚动条高度 用于计算在出现滚动条的时候滑动到底部可添加插件到可视区范围内 | Number  | 0                              |
| throttle-delay       | 节流间隙时长(拖拽或缩放时，间隔多久触发一次响应事件 单位ms)  | Number  | 10                             |



#### **slot 插槽:** 

| slot插槽名称 | 描述       |      |      |
| ------------ | ---------- | ---- | ---- |
| content      | 拖拽内容区 |      |      |

 

#### **event事件：**

| event事件名称 | 描述                  | 类型             | 参数                                                         |
| ------------- | --------------------- | ---------------- | ------------------------------------------------------------ |
| click         | 点击拖拽内容区触发    | function(id) {}  | (id) type:String 点击的当前拖拽外壳唯一标识                  |
| delete        | 点击删除按钮触发      | function(id) {}  | (id) type:String 点击的当前拖拽外壳唯一标识                  |
| change        | 位置/大小发生变化触发 | function(obj) {} | (obj) type:Object{id,width,height,left,top} 当前变动的拖拽外壳基本信息 |

​			 

#### **实例获取**

 1. 通过绑定ref 获取组件实例

    `this.$refs[id].getAttribute()`

 2. 通过id 获取组件实例 （小程序）

    `this.selectComponent("#id").$vm.getAttribute()`

    getAttribute(){return:Obj}返回当前组件的一些基础信息：

    ​	`Obj: {
	​	​		id,
	​	​		width,
	​	​		height,
	​	​		left,
	​	​		top
    ​		}`

#### **使用案例**

参考demo：[https://github.com/CrazyAz10/az-drag-temp-uni-wx](https://github.com/CrazyAz10/az-drag-temp-uni-wx)

以下案例并非完整代码 请自行参考使用

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



**注释：插件还在更新迭代中，有问题可以评论区留言，不定时间会扫荡评论区**