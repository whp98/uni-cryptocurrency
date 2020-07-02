<template>
	<view>
		<view style="display: flex;width: 100%;flex-direction:row;justify-content: center;">{{coin}}实时价格折线图</view>
		<view class="qiun-charts">
			<!--#ifdef MP-ALIPAY -->
			<canvas canvas-id="canvasLineA" id="canvasLineA" class="charts" :width="cWidth*pixelRatio" :height="cHeight*pixelRatio"
			 :style="{'width':cWidth+'px','height':cHeight+'px'}" @touchstart="touchLineA"></canvas>
			<!--#endif-->
			<!--#ifndef MP-ALIPAY -->
			<canvas canvas-id="canvasLineA" id="canvasLineA" class="charts" @touchstart="touchLineA"></canvas>
			<!--#endif-->
		</view>
	</view>
</template>

<script>
	import uCharts from '@/components/u-charts/u-charts.js';
	var _self;
	var canvaLineA = null;
	export default {
		data() {
			return {
				cWidth: '',
				cHeight: '',
				pixelRatio: 1,
				textarea: '',
				pagesize: 10,
				line: {
					categories: [],
					series: [{
						name: "",
						data: []
					}]
				},
				currentPrice: "",
				coin: "bitcoin",
				timer: '',
				flag:false,
				timeforflush:1000
			}
		},
		onHide() {
			clearInterval(this.timer);
		},
		onShow(){
			this.timer = setInterval(this.changeData, this.timeforflush);
		},
		onDestroy(){
			clearInterval(this.timer);
		},
		onLoad(e) {
			if(e.coin){
				this.coin=e.coin;
			}
			_self = this;
			//#ifdef MP-ALIPAY
			uni.getSystemInfo({
				success: function(res) {
					if (res.pixelRatio > 1) {
						//正常这里给2就行，如果pixelRatio=3性能会降低一点
						//_self.pixelRatio =res.pixelRatio;
						_self.pixelRatio = 2;
					}
				}
			});
			//#endif
			this.cWidth = uni.upx2px(750);
			this.cHeight = uni.upx2px(500);
			this.getServerData();
			_self.showLineA("canvasLineA", this.line)
			// this.timer = setInterval(this.changeData, this.timeforflush);
			
		},
		methods: {
			getServerData() {
				var _this = this;
				uni.closeSocket({});
				uni.connectSocket({
					url: 'wss://ws.coincap.io/prices?assets=' + this.coin
				});
				uni.onSocketMessage(function(res) {
					var obj = JSON.parse(res.data)
					for (let key in obj) {
						if (key === _this.coin) {
							_this.currentPrice = parseFloat(obj[key]).toFixed(3);
						}
					}
					if(!_this.flag){_this.flag=true;}
				});
			},
			showLineA(canvasId, chartData) {
				canvaLineA = new uCharts({
					$this: _self,
					canvasId: canvasId,
					type: 'line',
					fontSize: 11,
					animation: false,
					padding: [15, 20, 0, 15],
					legend: {
						show: true,
						padding: 5,
						lineHeight: 11,
						margin: 0,
					},
					dataLabel: true,
					dataPointShape: true,
					background: '#FFFFFF',
					pixelRatio: _self.pixelRatio,
					categories: chartData.categories,
					series: chartData.series,
					animation: true,
					xAxis: {
						type: 'grid',
						gridColor: '#CCCCCC',
						gridType: 'dash',
						dashLength: 8,
						boundaryGap: 'justify',
						format: (val) => {
							return val.toFixed(0)+'秒'
						}
					},
					yAxis: {
						gridType: 'dash',
						gridColor: '#CCCCCC',
						dashLength: 8,
						splitNumber: 5,
						format: (val) => {
							return val.toFixed(2) + '美元'
						}
					},
					width: _self.cWidth * _self.pixelRatio,
					height: _self.cHeight * _self.pixelRatio,
					extra: {
						line: {
							type: 'curve'
						}
					}
				});
				//下面是默认选中索引
				let cindex = 3;
				//下面是自定义文案
				let textList = [{
					text: '我是一个标题',
					color: null
				}, {
					text: '自定义1：值1',
					color: '#2fc25b'
				}, {
					text: '自定义2：值2',
					color: '#facc14'
				}, {
					text: '自定义3：值3',
					color: '#f04864'
				}];
				//下面是event的模拟,tooltip的Y坐标值通过这个mp.changedTouches[0].y控制
				let tmpevent = {
					mp: {
						changedTouches: [{
							x: 0,
							y: 80
						}]
					}
				};
				setTimeout(() => {
					canvaLineA.showToolTip(tmpevent, {
						index: cindex,
						textList: textList
					});
				}, 200)
			},
			touchLineA(e) {
				canvaLineA.touchLegend(e);
				canvaLineA.showToolTip(e, {
					format: function(item, category) {
						return category + ' ' + item.name + ':' + item.data
					}
				});
			},
			changeData() {
				if(this.flag){
					console.debug("hello");
					console.debug(this.line.categories)
					console.debug(this.line.series[0].data)
					if (this.line.categories.length > this.pagesize) {
						this.line.categories.shift();
						this.line.series[0].data.shift();
						var time = new Date();
						var str_time = time.getSeconds();
						this.line.categories.push(str_time);
						this.line.series[0].data.push(this.currentPrice);
						console.debug(3)
						
					} else {
						if (this.line.series.size < 1) {
							console.debug(2)
							
							this.line.series[0]["data"] = [];
							this.line.series[0]["name"] = this.coin;
						} else {
							console.debug(1)
							var time = new Date();
							var str_time = time.getSeconds();
							this.line.categories.push(str_time);
							this.line.series[0].data.push(this.currentPrice);
						}
					}
					canvaLineA.updateData({
						series: this.line.series,
						categories: this.line.categories,
						animation: false
					});
				}
			},
			testserver(){
				console.debug("hello")
			}
		}
	}
</script>

<style>
	/*样式的width和height一定要与定义的cWidth和cHeight相对应*/
	.qiun-charts {
		width: 750upx;
		height: 500upx;
		background-color: #FFFFFF;
	}

	.charts {
		width: 750upx;
		height: 500upx;
		background-color: #FFFFFF;
	}
</style>
